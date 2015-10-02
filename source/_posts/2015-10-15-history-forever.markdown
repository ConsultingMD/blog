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

# Make History last forever - keep history in folders by Y/M/Day_host
# Thanks to - https://twitter.com/michaelhoffman https://twitter.com/michaelhoffman/status/639178145673932800

Make sure that the `.history` folder exists and make sure that the year/month folders are created.

> NOTE: For some these folders may get created automagically when declaring their HISTFILE.  I kept seeing errors with this under OSX Yosemite so added the belt to the suspenders

```bash
mkdir -p "${HOME}/.history/$(date -u +%Y/%m/)"
```


