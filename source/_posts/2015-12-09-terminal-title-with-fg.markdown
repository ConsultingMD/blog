---
layout: post
title: "Bash: get a nice, dynamic terminal title"
date: 2015-12-09 16:11:42 -0800
comments: true
categories: bash, tips
author: Jesse Dhillon
---
I'll explain here how to have your terminal's title set to the current
directory when you're at the prompt, and the currently running command if there
is a job running. Additionally, I'll show how to wrap the `fg` built-in to
set the right title when you resume suspended jobs.

# Missed it by _that_ much

Most bash users who've looked into updating their terminal title with the
currently running command have probably come across this line:

```
trap 'echo -ne "\e]0;${BASH_COMMAND}\007"' DEBUG
```

The `trap` with `DEBUG` exposes a hook in bash on each command, and it
exposes the command name through the `BASH_COMMAND` variable. This allows us to
set the terminal title, which is achieved by sending the correct ANSI escape
sequence. (These control characters are specific to xterm, and are not part of
the normal terminal control set, which are used to *e.g.* move the cursor.)

This is a great first step, but there are a few problems if you stop right
here. First, if you already have a `PROMPT_COMMAND` then the name of the
terminal will be the command named in your `PROMPT_COMMAND`, in my case,
a function named `_update_ps1` -- that's hardly useful. I'd rather see
the current working directory when I'm at the shell -- that's stored in the
`PWD` variable.

Second, if you use [job control](http://web.mit.edu/gnu/doc/html/features_5.html)
a lot, like I do, then when you resume a suspended job, the name in
`BASH_COMMAND` will be `fg` and not the name of the resumed job.  That is
totally useless if you're using terminal titles, as after you've resumed jobs
in multiple terminal windows (or panes in tmux) their names will all be `fg`.

# The solution: write a better trap handler

First, we'll write a function to be called on each command:

```
_update_title () {
    if [ "$BASH_COMMAND" == '_update_ps1' ]; then
        title=$PWD
    elif [ "$1" ]; then
        title=$@
    else
        title=$BASH_COMMAND
    fi
    printf "\e]0;%s\007" "$title"
}
```

(_N.B._ the quotes around `$title` at line 9 are important, to force bash to
apply the full command as one argument, even when it contains spaces.)

The basic logic is this:

- if there is a `BASH_COMMAND` defined (*i.e.* we have trapped a command), and
  it is the name of our `PROMPT_COMMAND` function, then we know we are idling
  at the prompt; set our title to `PWD`
- otherwise, if this function has arguments, set our title to the concatenation
  of all our arguments
- lastly, if we do have a `BASH_COMMAND` and it's not our `PROMPT_COMMAND`, use
  that as our title

Having defined this function, we install our trap handler in the usual way:

```
trap '_update_title' DEBUG
```

# ...and alias fg

Now, we have the basic functionality that we desire, but the problem with `fg`
still remains. Let's write a wrapper around `fg` which sets the right command
name.

```
foreground() {
    extglob_state=$(shopt -p extglob)
    shopt -s extglob
    case "$1" in
        [0-9]*([0-9]))
            jobname=$(jobs -l | grep -e "^\[$1\]" | awk '{$1=$2=$3=""; print $0}');;
        +|-)
            jobname=$(jobs -l | grep -e "^\[[[:digit:]]\+\]$1" | awk '{$1=$2=$3=""; print $0}');;
        *)
            jobname=$(jobs -l | grep -e "^\[[[:digit:]]\+\]+" | awk '{$1=$2=$3=""; print $0}');;
    esac
    _update_title ${jobname##*( )}
    \fg $1
    $extglob_state
}
alias fg=foreground
```

(_N.B._ we need the `extglob` option to give us [extended glob pattern matching](http://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html#Pattern-Matching)
in our `case` statement, but we should [be a Boy Scout](http://www.scouting.org/scoutsource/BoyScouts/AdvancementandAwards/joining.aspx)
and restore the state of any options we change in our function.)

`fg` takes an optional argument. If that argument is a digit, it is interpreted
as a job number, and the corresponding job is switched to the foreground. (You
can list all jobs with the `jobs` command.) If the argument is not a digit,
there are two other correct inputs:

- `+` which refers to the most recently suspended job (the "current" job)
- `-` which refers to the next most recently suspended job (the "previous" job)

If there's no argument, the the default behavior is `fg +`. What our alias
function does is parse the arguments, get the corresponding job name from the
output of `jobs`, and sets the title using our previous `_update_title`
function. Then, we actually invoke the true `fg` built-in; since we've aliased
the name `fg`, we have to use an escape, `\fg`, to refer to the unaliased
instance.

We could just echo the terminal escape sequence from this function, but by
reusing our previous function, we can have consistent behavior if we modify
our terminal title format *e.g.* to prepend `HOSTNAME` or `USER` to our title:

```
    printf "\e]0;%s@%s:%s\007" $USER $HOSTNAME "$title"
```

# Bonus: tmux

If you're like me, you use tmux to manage multiple terminal sessions within
one window of your terminal emulator. In tmux, the terminal title is set per
pane -- so if you have one terminal window which is split into multiple panes,
each one will have its title set.

In our environment at GR, we often have multiple services running as part of
our development environment. I keep those running in one tmux window, split
into multiple panes. Now, each pane gets the title of the running service. As
a bonus, here's how to get the terminal title into the tmux window list:

```
window-status-current-format ' #I #W (#T) '
```

The `#T` expands to the current terminal title.

# Final advice

If you're going to be playing around with `trap`, you should always uninstall
any existing handlers before you switch to a new handler:

```
trap - DEBUG
```
