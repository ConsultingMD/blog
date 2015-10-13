---
layout: post
title: "History is Forever"
date: 2015-10-15 16:03:22 -0700
comments: true
categories:
author: Kevin Moore

---

The history of your command-line can be an invaluable resource to look back through and figure out how you made that magic happen that one time you cobbled together the right incantation to successfully pipe the `sed` of the `awk` of your `grep` .  It can also serve as a great starting point as a recipe for automation.  

We know we should all start with our automation tools, but sometimes it is easier or more expedient just to get that new server up and running and tweaked and working with a bunch of command-line-fu.  Other times it is new territory and you need to so some trial and error, or experimentation before you're ready to automate.  Once you do have things working, that history file is a fantastic starting place to use to go back and write those puppet modules, or create a script or utility to capture and make repeatable what you've just done. Too many times I've had multiple bash sessions open or killed a terminal session without thinking and had that history lost or clobbered.

Somewhere along the way I stumbled across a tweet from [@michaelhoffman](https://twitter.com/michaelhoffman https://twitter.com/michaelhoffman/status/639178145673932800) for a great method of handling history files to make sure you always have the full history for every terminal session you ever have.

##[The Short Short Version:](https://en.wikiquote.org/wiki/Spaceballs#Dialogue)

Edit your `.bash_profile`

Paste in the following:

```bash
mkdir -p "${HOME}/.history/$(date -u +%Y/%m/)"
HOSTNAME_SHORT=`hostname -s`
export PROMPT_COMMAND='history -a'
HISTFILE="${HOME}/.history/$(date -u +%Y/%m/%d.%H.%M.%S)_${HOSTNAME_SHORT}_$$"
HISTFILESIZE=10000000

histgrep () {
    grep -r "$@" ~/.history
    history | grep "$@"
}
export -f histgrep
```
You're welcome

##The Long Version:

Rather than a single monolithic history file that gets clobbered when we have multiple terminal sessions, we'll declare our `$HISTFILE` as a collection of files sorted into folders and sessions.  This way we retain an individual history for all of our sessions and we can actually capture this info across multiple workstations or servers and rsync that back and forth if we truly want to have one-history-to-rule-them-all.


###Now we'll do the deep dive and explain each command in detail:


   First we make sure that the `.history` folder exists in our home directory and ensure that the current `year/month/` folder exists or gets created.

> NOTE: For some, these folders may get created automagically when declaring their HISTFILE to include a folder path.  I kept seeing errors with this under OSX Yosemite so added this belt to the suspenders

```bash
mkdir -p "${HOME}/.history/$(date -u +%Y/%m/)"
```

We don't want to see a lot of `my-system.some-sub-domain.some-long-ass-domain.io` or more likely `myhostname.local` in our history filenames so we make sure we have just the basic hostname

```bash
HOSTNAME_SHORT=`hostname -s`
```
In case our shell session ends abruptly we want to make sure things are making it to history as we go.

```bash
export PROMPT_COMMAND='history -a'
```
Okay this is where the real magic comes in.  Again, all credit to [@michaelhoffman](https://twitter.com/michaelhoffman https://twitter.com/michaelhoffman/status/639178145673932800) for this next bit.  We set our history filename in the format `Year/Month/Day.Hour.Minute.second_hostname_ShellProcessID`  The result will be a hierarchical folder tree with our history broken out into individual files by session and grouped into folders by month and months rolled up into folders by year.
 


```bash
HISTFILE="${HOME}/.history/$(date -u +%Y/%m/%d.%H.%M.%S)_${HOSTNAME_SHORT}_$$"
```

The result is that we now have a folder of history files rather than a singular file.  It also means that when we open 5 different terminal windows, each retains their own history and when the shell session ends no session clobbers the history of another.

```bash
MyMac:~/.history$ ls -l
total 0
drwxr-xr-x   5 someguy  staff   170 Sep 30 17:00 2015
MyMac:~/.history$ cd 2015/
MyMac:~/.history/2015$ ls -l
total 0
drwxr-xr-x   4 someguy  staff  136 Aug 31 18:44 08
drwxr-xr-x  26 someguy  staff  884 Sep 30 15:01 09
drwxr-xr-x  19 someguy  staff  646 Oct 13 14:40 10
MyMac:~/.history/2015 cd 09
MyMac:~/.history/2015/09$ ls -l
total 672
-rw-------   1 someguy  staff     312 Sep 18 18:53 19.01.37.31_MyMac-2_401
-rw-------   1 someguy  staff     281 Sep 18 18:52 19.01.47.56_MyMac-2_3811
-rw-------   1 someguy  staff     385 Sep 18 18:52 19.01.50.33_MyMac-2_4050
-rw-------   1 someguy  staff     168 Sep 18 18:52 19.01.52.18_MyMac-2_4547
-rw-------   1 someguy  staff      84 Sep 18 18:54 19.01.52.56_MyMac-2_4828
-rw-------   1 someguy  staff      94 Sep 18 19:13 19.01.54.10_MyMac-2_5009
-rw-------   1 someguy  staff      79 Sep 18 19:11 19.02.07.26_MyMac_5227
-rw-------   1 someguy  staff     451 Sep 18 19:26 19.02.26.20_MyMac_5413
-rw-------   1 someguy  staff      35 Sep 18 19:26 19.02.26.32_MyMac_5800
...
```

Now that we have our history spread across a bunch of files we want an easy way to be able to search back through our current history buffer and all of the individual history files.  `histgrep` will recursively grep through our history files and then the current history buffer.


```bash
histgrep () {
    grep -i -r "$@" ~/.history
    history | grep -i "$@"
}
```

Now when we do some magic command-line-fu we know it will be around to find.  

###An Example:
A few weeks ago I wanted to get the IP addresses of some of our AWS web servers.  I wanted to make sure I was getting the staging servers and not the live instances.  I probably should have written this down somewhere or made it into a script, but I know it was just a quick (and a little ugly) one-liner.  I remember there was some magic around querying an elb so:

```bash
MyMac:~$ histgrep elb
/Users/someguy/.history/2015/09/22.02.25.48_MyMac_66471:subl ./elb-tool.py 
/Users/someguy/.history/2015/09/22.03.17.16_MyMac_68743: elb="www-staging-elb"; for machine in `aws elb describe-instance-health --load-balancer-name ${el} |grep InstanceId |cut -f2 -d":" |tr -d " ,\"" `; do aws ec2 describe-instances --instance-ids $machine |grep PublicIpAddress |cut -f2 -d":"| tr -d " ,\""; done
   11  histgrep elb
```   

Voila.  We found our magic incantation.  We also know what session that came from so we can now go back to `22.03.17.16_MyMac_68743` and maybe find some of the other work from that session. 

I've only recently started using this trick but it has already proved to be massively useful.  Knowing that I'm no longer losing history info on a regular basis and having a context around when and where a command in my history was actually run has been a welcome side benefit. 


