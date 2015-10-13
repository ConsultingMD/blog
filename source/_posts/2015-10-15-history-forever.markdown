---
layout: post
title: "History is Forever"
date: 2015-10-15 16:03:22 -0700
comments: true
categories:
author:

---

tl;dr

Edit your `.bash_profile`

Paste the following:

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
The Long of it:


Make sure that the `.history` folder exists in our home and make sure that the year/month/ folders are created.

> NOTE: For some these folders may get created automagically when declaring their HISTFILE.  I kept seeing errors with this under OSX Yosemite so added the belt to the suspenders

```bash
mkdir -p "${HOME}/.history/$(date -u +%Y/%m/)"
```

We don't want to see a lot of `my-system.some-sub-domain.some-long-ass-domain.io` or more likely `myhostname.local` 

```bash
HOSTNAME_SHORT=`hostname -s`
```
In case our shell session ends abruptly we want to make sure things are making it to history as we go.

```bash
export PROMPT_COMMAND='history -a'
```
Okay this is where the real magic comes in.  All credit to [@michaelhoffman](https://twitter.com/michaelhoffman https://twitter.com/michaelhoffman/status/639178145673932800) for this next bit.  We set our history filename in the format Year/Month/Day.Hour.Minute.second\_hostname\_ShellProcessID
 


```bash
HISTFILE="${HOME}/.history/$(date -u +%Y/%m/%d.%H.%M.%S)_${HOSTNAME_SHORT}_$$"
```

The result is that we now have a folder of history files rather than a singluar file.  It also means that when we open 5 different shell windows each retains their own history and when the shell session ends no one session clobbers the history of another.

```bash

bash-3.2$ cd .history/2015/09/
bash-3.2$ ls -al
total 672
drwxr-xr-x  26 someguy  staff     884 Sep 30 15:01 .
drwxr-xr-x   5 someguy  staff     170 Sep 30 17:00 ..
-rw-------   1 someguy  staff     312 Sep 18 18:53 19.01.37.31_MyMac-2.local_401
-rw-------   1 someguy  staff     281 Sep 18 18:52 19.01.47.56_MyMac-2.local_3811
-rw-------   1 someguy  staff     385 Sep 18 18:52 19.01.50.33_MyMac-2.local_4050
-rw-------   1 someguy  staff     168 Sep 18 18:52 19.01.52.18_MyMac-2.local_4547
-rw-------   1 someguy  staff      84 Sep 18 18:54 19.01.52.56_MyMac-2.local_4828
-rw-------   1 someguy  staff      94 Sep 18 19:13 19.01.54.10_MyMac-2.local_5009
-rw-------   1 someguy  staff      79 Sep 18 19:11 19.02.07.26_MyMac.local_5227
-rw-------   1 someguy  staff     451 Sep 18 19:26 19.02.26.20_MyMac_5413
-rw-------   1 someguy  staff      35 Sep 18 19:26 19.02.26.32_MyMac_5800

```

Now that we have our history spread across a bunch of files we want an easy way to be able to search back through our current history buffer and all of the individual history files.  This will recursively grep through our history files and then the current history buffer.


```bash
histgrep () {
    grep -i -r "$@" ~/.history
    history | grep -i "$@"
}
```