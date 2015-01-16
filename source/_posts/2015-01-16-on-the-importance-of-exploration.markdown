---
layout: post
title: "On the Importance of Exploration"
date: 2015-01-16 10:52:32 -0800
comments: true
categories: tools
author: Alex Bird
---

If I was into wishing I would wish there was a way to directly hook my brain up to a computer so I didn't have to use a keyboard and mouse all day. "They" will probably come up with that at some point, but for now I'm stuck with the current offerings. Fortunately, some of the tools and shortcuts available to developers are simply delightful. I'd like to share some of the ones I've discovered recently, and hopefully encourage you to explore for yourself.

There is a balance to be found between customizing everything and utilizing every shortcut, and learning a system's standard tools. It takes a lot of experimenting to figure out a good set of tools -- editor, shell, etc, and it's well worth it, but you'd do better by not customizing everything into some pimp-my-ride monstrocity. You won't be able to remember all the crap you have setup anyway. Nobody else will have any idea how to use your system and similarly, you will feel disabled when you log in to servers or pair on someone else's machine. It's important to keep exploring better ways of doing things, but don't go crazy.

The mental overhead of adopting a new workflow is considerable. It's best to try one or two things at a time until you pick it up. The goal is to get to the point where you don't have to think about using the tool, plugin, shortcut, etc. Although, I suspect that code quality has little to do with how fast you type, or how few keys you press, and everything to do with thinking about what you're doing. We get used to what we know, and the familiarity of that is just easier than learning a new way, even if it's faster and more badass.


### Basics

No matter what tools you use, having a faster key repeat and delay speed will be better. If you're used to slower speeds, that's okay. You will also get used to faster speeds. You can ease into faster settings by slowly ramping up until you find your sweet spot. Of course it will feel weird at first, but it's one of those things where if you can get used to it, you'll cringe when you have to go back. The fastest OSX defaults are artificially limited to not-fast-enough values, but you can increase them more with a tool called <a href="https://pqrs.org/osx/karabiner/" target="_blank">Karabiner</a>.

Tools that rely heavily on `CTRL` and `ESC` can benefit from the following: remap left `CTRL` to `CAPS LOCK`. The trade-off of not having a `CAPS LOCK` key is outweighed by having `CTRL` significantly more accessible. Even better, you can map left `CTRL` to `CAPS LOCK` _only_ when it is pressed in combination with another key. When pressed alone it becomes `ESC`. This may sound wacky, but it's just simple enough for your muscle memory to actually pick it up and gets rid of a lot of unnecessary reaching. It also doesn't affect the usability of your keyboard for those unfamiliar with your setup. (also see Karabiner for OSX, and xcape for Linux)

Linux example:

```
# in .bashrc
# remap caps lock to control
setxkbmap -option 'caps:ctrl_modifier'

# make caps lock be esc when pressed alone
killall xcape &> /dev/null
~/bin/xcape -e 'Caps_Lock=Escape'
```

Finally, a clipboard manager will keep track of your copy+paste history. Highly recommended. ([Alfred](http://www.alfredapp.com/) for OSX, [Parcellite](http://parcellite.sourceforge.net/?page_id=10) for Linux)


### Editor

The editor is really one of the most important of the programmer's tools, and it doesn't really matter which one you use. Emacs has VIM mode, Sublime has Emacs mode and VIM mode, VIM has Emacs mode and lots of VIM modes (ha). None of that makes any sense. Even so, I highly recommend trying to figure out at least one of the classic popular editors: VIM and Emacs. They both go very deep. Learning one of these tools gives you something akin to lifelong friend that you keep discovering more about as the years go by.

Lately, I've been all about search as my primary way to navigate code, and any editor worth using should support search in various capacities. Use search generously to open files, open project-wide code locations, and jump around to locations within a file.

For smaller movements, however, there are so many possibilities of cursor movement that it's impossible to not have to use the arrow keys or a cumbersome equivalent. I don't like the arrow keys, and I don't like you if you use them (just kidding). The world beyond arrow keys is wonderful. If you've ever wondered why someone would bother learning something as arcane as VIM, one of the main reasons is that it provides so many efficient ways to make local-level cursor movements. There are so many that it's literally impossible to know about them all, and you sometimes find yourself rediscovering them without being aware of having known them before, but yet also having a stirring in your deep memory of something old and familiar like events from a past life.

Searching for small movements is then possible too. EasyMotion is a plugin that lets you jump to any visible character in three keystrokes. (EasyMotion: VIM - https://github.com/Lokaltog/vim-easymotion, Emacs - http://www.emacswiki.org/emacs/AceJump, Sublime - https://github.com/tednaleid/sublime-EasyMotion)

Press the EasyMotion activation key, my my case `s`:

![Activate EasyMotion](http://grh-wiki-pics.s3.amazonaws.com/bird-easymotion-1.png)

This brings up a prompt so type the first character of what you want to search for, in this case `f`. Then type the red character corresponding to the `f` you want to jump to, in this case `k`:

![EasyMotion search](http://grh-wiki-pics.s3.amazonaws.com/bird-easymotion-2.png)

The cursor jumps:

![EasyMotion jump](http://grh-wiki-pics.s3.amazonaws.com/bird-easymotion-3.png)

I'm not saying it's bad to just move around with arrow keys, but that there is enlightenment to be found beyond it. If you can't seamlessly search throughout these various scopes, it's worth getting that setup for your editor.


### Shell

Bash has a ton of functionality out of the box and it feels great to be able to log into servers with a completely standard Bash without losing productivity. There are lots of great shortcuts to learn: http://www.skorks.com/2009/09/bash-shortcuts-for-maximum-productivity/. One not mentioned there that I've been using a lot is `CTRL-d` at an empty prompt, which ends your current login, and potentially closes the tab/window/remote session.


Here are some other Bash goodies:

```
# in .bashrc
# search through history with up/down arrows
bind '"\e[A": history-search-backward'
bind '"\e[B": history-search-forward'"]]"'
```

!$ expands to the last argument of the last command:

```
bird@bird-840: /tmp$ echo hi > foo.txt
bird@bird-840: /tmp$ cat !$
cat foo.txt
hi
```


Autojump (https://github.com/joelthelion/autojump), although not really related to Bash, is a tool that lets you quickly cd to your most used directories. It keeps track of which directories you cd to, and automatically jumps you to your most used directories with the fewest characters possible:

```
bird@bird-840: /tmp/$ ls
fluffhead/
bird@bird-840: /tmp/$ j fl
/tmp/fluffhead
bird@bird-840: /tmp/fluffhead$
```


`pushd` and `popd` are built-in Bash commands like cd, but they maintain a stack of your directories:

```
bird@bird-840: /tmp$ pushd fluffhead/
/tmp/fluffhead /tmp
bird@bird-840: /tmp/fluffhead$ pushd ~
~ /tmp/fluffhead /tmp
bird@bird-840: ~ $ popd
/tmp/fluffhead /tmp
bird@bird-840: /tmp/fluffhead$ popd
/tmp
bird@bird-840: /tmp$
```


Aliases are super useful:

```
# copy/paste (linux specific)
#
# eg: cat foo.txt | copy
#     paste > bar.txt
alias paste='xsel -b'
alias copy='xsel -ib'

# git
alias gb='git branch'
alias gci='git commit'
alias gco='git checkout'
alias gd='git diff'
# show diff of what has been added to the index
alias gdc='git diff --cached'
# see git alias below
alias gl='git lg'
# show all the project's commits, not just your current branch
alias gla='git lg --branches --remotes --tags'
alias gs='git status'

# ls
alias ls='ls -F --color=auto'
alias l='ls'
alias la='ls -Al'
alias ll='ls -ltrh'

# misc
alias genpass='< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c16; echo'
alias grep='grep --color=auto'
```

It's hard to beat `find . | grep foo.txt` for navigating the file system from the command line. Copy and paste whenever possible, as opposed to retyping things.


### Git

There are some handy git aliases as well for your .gitconfig, which work in tandem with the above Bash aliases:

```
[alias]
  br = branch
  ci = commit
  co = checkout
  d  = diff
# show the commit graph
  lg = log --graph
  st = status
```

Use a global `.gitignore` file:

```
[core]
  excludesfile = /home/bird/.gitignore_global
```

Stop typing `git push origin master`. You can just `git push` or `git pull` to/from the remote branch of the same name with these settings:

```
# From the git docs:
#
# upstream - push the current branch to its upstream branch. With this, git
# push will update the same remote ref as the one which is merged by git pull,
# making push and pull symmetrical. See "branch.<name>.merge" for how to
# configure the upstream branch.
#
# simple - like upstream, but refuses to push if the upstream branch’s name is
# different from the local one. This is the safest option and is well-suited
# for beginners. It will become the default in Git 2.0.
#
# current - push the current branch to a branch of the same name.
[push]
  default = current
[pull]
  default = simple
```


### Conclusion

These are just some things I've discovered recently and have been trying to adopt into my workflow. The point isn't that these are the best tools or shortcuts, it's that exploration can lead you to discover new things that make your job easier. Something is true only until something more true comes along to replace it, which is why it's vital to experiment with different ways of doing your work. The right tools can turn work into fun, which is something you can't put a price on.
