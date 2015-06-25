---
layout: post
title: "Securely Transmitting Credentials from the CLI with OSX &amp; Gmail"
date: 2015-06-19 15:35:37 -0700
comments: true
categories: CLI OSX Postfix GPG
author: Kevin Moore

---

## Overview

As an IT administrator, I often have the need to create new accounts for users on various systems.  Not all of our services are setup for, or capable of, SSO.  Because of this, I need to be able to transmit these login credentials securely whether the user is in our office, in a satellite office, or working remotely.

To make sure we can easily send credentials and lots of other useful files from the command line, we'll setup the OSX native postfix mail to use our [Google Apps for Work](https://www.google.com/work/apps/business/) account as our mail relay.  We'll connect to gmail's SMTP via port 587 and use STARTTLS so that we know we have a secure connection to the server. Once we have our email setup we'll add a shell function that easily allows us to encrypt and send a file via email in a single step.

> **NOTE:** I've left the setup of GPG out of the scope of this document, largely because the DMG package installer of the [GPG Tools Suite](https://gpgtools.org/) does pretty much everything for you and has good documentation on how to create and publish your keys.

## Setting Up Postfix

We'll start by making some configuration changes to postfix to setup gmail as our relay server and to configure postfix to authenticate and use TLS.  Here are the pertinent changes / additional lines in our `/etc/postfix/main.cf` to make this work:

```bash
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

We now need to create the sasl_passwd file:

> **NOTE:** _In our domain, we require 2 Factor authentication for all users; so our sasl password will be an [Application Specific Password](https://support.google.com/accounts/answer/185833?hl=en) from google and will be used specifically for sending from postfix.  If you're not using 2FA (and you really  should be) then you'll use your main account password._

```bash
echo "[smtp.gmail.com]:587 user_name@your_domain.com:YOUR_PASSWORD" > /etc/postfix/sasl_passwd
```

We want to lock down our password file so that only we have access to it.  Then run postmap to re-index/reload the file.  We'll also restart postfix to make sure all of the settings are reloaded.

```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd

sudo postfix stop
sudo postfix start
```

Postfix is now configured and we're ready to start sending some email from the command line.

## Sending A Test Message

For troubleshooting, we'll open a second terminal window and tail the postfix log so we can see what's going on and can look for errors

```bash
sudo tail -f /var/log/mail.log
```

Now we can send some test messages to ourselves from the command line to see if we have everything configured correctly:

```bash
MyMac:~$ mail -s "Test message 1" me@mydomain.com
My Fancy Test message body
Then a blank line and then a CTRL-D to end


EOT
```

In our other window we should see some logging coming through:

```bash
Jun 19 15:57:11 MyMac.local postfix/master[69038]: daemon started -- version 2.11.0, configuration /etc/postfix
Jun 19 15:57:11 MyMac.local postfix/pickup[69039]: 910AEC86481: uid=503 from=<Me>
Jun 19 15:57:11 MyMac.local postfix/cleanup[69041]: 910AEC86481: message-id=<20150619218700.910AEC86481@smtp.gmail.com>
Jun 19 15:57:11 MyMac.local postfix/qmgr[69040]: 910AEC86481: from=<Me@MyDomain.com>, size=316, nrcpt=1 (queue active)
Jun 19 15:57:13 MyMac.local postfix/smtp[69043]: 910AEC86481: to=<me@mydomain.com>, relay=smtp.gmail.com[74.125.20.108]:587, delay=2.8, delays=0.64/0.09/0.94/1.1, dsn=2.0.0, status=sent (250 2.0.0 OK 1434754632 he9sm12236175pbc.7 - gsmtp)
Jun 19 15:57:13 MyMac.local postfix/qmgr[69040]: 910AEC86481: removed
Jun 19 15:58:11 MyMac.local postfix/master[69038]: master exit time has arrived

```

If you see `status=sent` then you're golden.  If you're sending this message to yourself as a test, make sure you check your Sent-Items for the message.  Gmail automatically puts everything from "You" in your sent items even if it is also To "You"

## Troubleshooting Authentication

If something went wrong and the message didn't go through, or you see errors in the Postfix log; check the mail queue to see if messages are going out or are getting stuck in queue with: `sudo mailq`  If you are seeing messages queued up, you can attempt to flush the queue with: `sudo postfix flush`

If you do have a config or authentication error, chances are you've now queued up a bunch of messages and want to get a clean slate so you can try your next test.  You can delete everything in queue with `sudo postsuper -d ALL`

If you are seeing errors in your logging we can do some other checks to see if it is an authentication issue or some other config problem.

First we use openssl to start a TLS session to gmail's SMTP server:

```bash
openssl s_client -starttls smtp -connect smtp.gmail.com:587
```

Now we say ["hello"](http://www.samlogic.net/articles/smtp-commands-reference.htm) and let the server know our hostname (localhost is fine):

```bash
EHLO localhost
```

We should  get back something like:

```bash
250-mx.google.com at your service, [128.177.170.122]
250-SIZE 35882577
250-8BITMIME
250-AUTH LOGIN PLAIN XOAUTH2 PLAIN-CLIENTTOKEN XOAUTH
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-CHUNKING
250 SMTPUTF8
```

We'll need to create a base64 encoded string with our username and password in order to authenticate.  Open a second terminal window and type the following:

```bash
echo -e "\000my_user@my_domain.com\000mypasswd" |base64 |pbcopy
```


You now have a null delimited base64 encoded string of your username and password on your clipboard.  Switch back in your first terminal window, type `AUTH PLAIN ` and then paste your clipboard with `CMD+V`

```bash
AUTH PLAIN AG15X3VzZXJAbXlfZG9tYWluLmNvbQBteXBhc3N3ZAo=
```

If you see this error:

```bash
535-5.7.8 Username and Password not accepted. Learn more at
535 5.7.8 https://support.google.com/mail/answer/14257 qa1sm19325903pab.0 - gsmtp
```

Then you have an issue with your credentials and will need to verify your username and password.  Keep testing until you achieve success:

```bash
235 2.7.0 Accepted
```

Once you know your credentials are accepted you can type `CTRL-D` to end your session.  We now know that our credentials are good and that we're able to successfully establish a TLS session to the gmail SMTP server.

## Sending Credentials

Now that we can send mail from the command line we want to be able to use this to securely send files to others.  Whether we use the GUI or do this from the command line it involves several steps:

Either from the command line:

```bash
~$ gpg -e -r me@mydomain.com -r recipient@theirdomain.com --trust-model always foo.txt
~$ uuencode foo.txt.gpg foo.txt.gpg | mail -b me@mydomain.com -s "Your Encrypted Credentials" recipient@theirdomain.com
```
Or from the GUI:

> 1. Right click on the file:
>  1. Choose Services > OpenPGP: Encrypt File
>  1. Make sure to add the user's key and your own key.
> 1. Open up mail or the browser to send a message.
> 1. Attach the encrypted file.
> 1. Send the file.

This may not sound like a ton of steps but it's pretty easy to see that having to do this over and over gets old fast.  It is also easy to make an error and either send a credential to the wrong person (NBD here - they won't be able to open it) or worse accidentally attach the plaintext or original file instead of the encrypted version.


## Writing the Shell Function

To simplify all of this, we'll write a shell function called `send_credentials`.  This will wrap all of these steps into a single action that we can run from the command line.

We'll add the following function definition to `~/.bash_profile` so that `send_credentials` is always available in any of our terminal sessions:

```bash
my_email="user@my_domain.com"
function send_credentials {
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

export -f send_credentials
```

> Remember to re-source your .bash\_profile after making changes" `source ~/.bash_profile`

Our function takes 3 parameters:

> 1. The filename to encrypt
> 1. The email address of the user we want to send this to (This will also be used to identify the GPG key to be used for encryption)
> 1. A comment / subject line for the email

From there we create an encrypted copy of the file, adding our own and the recipient's keys and then email the encrypted copy of the file as an attachment.  I have this setup to BCC me with the email so that I have a record of what was sent out and confirmation that the message went through.

```bash
MyMac:~$ send_credentials secret_foo.txt recipient@domain.com "Super Secret Foo"
Emailing credentials to: recipient@domain.com
```

## Conclusion

Although it takes a few minutes to get setup and working, we can now easily send mail directly from the command line knowing that it will be safely relayed through google's servers, and we have the ability to securely send a credential to a user.

