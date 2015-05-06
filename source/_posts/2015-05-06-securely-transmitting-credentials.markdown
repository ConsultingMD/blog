---
layout: post
title: "Securely Transmitting Credentials from the commandline with OSX & gmail"
date: 2015-05-04 17:55:11 -0700
comments: true
categories: CLI commandline GPG
author: Kevin Moore

---

To make sure we can easily send these credentials and lots of other useful commandline things, we'll setup the OSX native postfix mail to use our Gmail apps for domains as the mail relay.  We'll connect to gmail's SMTP via port 587 and use STARTTLS so that we know we have a secure connection to the server.

Here are the pertinent changes / additional lines in /etc/postfix/main.cf to make this work.

```postfix
myhostname = smtp.gmail.com
mydomain = your_domain.com
myorigin = $mydomain

relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes

smtp_sasl_mechanism_filter = plain
```

we now need to create the sasl_passwd file

We require 2 Factor authentication for all users so our sasl password will be an [Application Specific Password](https://support.google.com/accounts/answer/185833?hl=en) from google specifically for your postfix sending.

```bash
echo "[smtp.gmail.com]:587 user_name@your_domain.com:YOUR_PASSWORD" > /etc/postfix/sasl_passwd
```

Lock down our password file so that only we have access to it, and use postmap to re-index/reload the file

```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd


sudo postfix stop
sudo postfix start
```

for troubleshooting - open a second terminal window and tail the postfix log so we can see what's going on and can look for errors

```bash
sudo tail -f /var/log/mail.log
```

Now we can send some test messages to ourselves from the commandline to see if we have everything configured correctly:


Check the mail queue to see if messages are going out or are getting stuck in queue.

sudo mailq

You can attempt to flush the mail queue with

sudo postfix flush


If you've queued up a bunch of messages during testing you can dump the queue

sudo postsuper -d ALL


It is hard to test TLS from the commandline but we can at least verify our username and password if you're getting authentication issues

openssl s_client -connect smtp.gmail.com:465

EHLO

echo "my_user@my_domain.com" |base64 |pbcopy

<Paste this into your terminal session>

echo "MY_SECURE_PASSWORD" |base64 |pbcopy

<Paste this into your terminal session>

I've added the following to my ~/.bash_profile so that I have the function `send_creds` available anywhere in my terminal session to encrypt and send a file to another user.

```bash
my_email="user@my_domain.com"
function send_creds {
    if [[ $# -ne 3 ]]; then
        echo "error: usage [filename user_email cred_type]"
        return 1
    fi
    file="$1"
    user_email="$2"
    cred_type="$3"

    gpg -q -e -r "${my_email}" -r "${user_email}" --trust-model always "${file}"
    if [[ $? -ne 0 ]]; then
        echo ""
        echo "ERROR - Encrypting credentials"
        echo ""
        return 2
    fi
    echo "Emailing encrypted credentials to: ${user_email}"
    uuencode ${file}.gpg ${file}.gpg | mail -b "${my_email}" -s "${cred_type} Credentials" ${user_email}
}

export -f send_creds
```
