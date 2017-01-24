---
layout: post
title: "Burying Your Dead Code"
date: 2017-01-24 13:15:12 -0800
comments: true
author: "Lindsey Anne <lindsey.hogg@grandrounds.com>"
categories: git bisect, dead code
---
As our code evolves and changes, little bits of leftovers from refactors or changed paths can build up as dead code. It's like the plaque that builds up on your teeth over time: not really noticeable at first, but after a trip to the dentist, you feel a whole lot better.

Removing dead code from your codebase is a simple process at its core: find the dead parts, prove that they're dead, and delete them. The tricky part is in that second step, but after a little trial and error, we discovered a couple tricks to speed up the process.

##### Start small

Before digging into a gigantic repo to see what you can rip out of there, consider the impact of what you're about to do and start with just a small piece of it. Since I was investigating one of our biggest repos, I decided to tackle a single Model first rather than, say, the entire lib directory.

##### Finding the dead parts

I'm sure there's other gems out there that perform similar assessment, but our favorite gem for this is [debride by Seattle Ruby Brigade](https://github.com/seattlerb/debride). It's not 100% perfect, but it will give you a good starting place to start digging around. The whitelisting option is particularly helpful after your initial investigation. Another option is the brute-force method, where you comb through a file and investigate each and every piece of it one by one. If your `debride` output is suspiciously large, this might be worth a try anyway.

##### Prove that they're dead

This is the fun part.

The specific service I was trying to clean up is one of our oldest repositories, and some of the functions that were showing up as dead ends had been sitting in there for months or even years. What's a girl to do, dig through hundreds and thousands of pull requests and commits until coming up with the right one? Oh no, we have a tool for that. It's called [git-bisect](https://git-scm.com/docs/git-bisect), and it's one of the coolest things I've learned so far about git.

`git bisect` is generally used for tracking down when a bug was introduced by using a binary search through as many commits as you'd like; you can start at the very beginning of your git history or somewhere in the middle. It checks out the repo at each commit, where you can test your issue and mark it as "good" or "bad." Rather than testing for a bug, though, I used it to `grep` for the method names that were appearing as dead from my previous step one at a time. If `grep` only gave one result, then nothing else was calling the method anymore and I'd mark it "bad." If it showed up with two or more results (minus tests that might still exist), I'd mark it "good." The point when searching for a bug is to find the last "good" commit before a bug was introduced; the point when using it to look for dead code is to track down when the consumer was removed.

##### Delete them

Eventually, I'd come across the commit where the diff showed the changed or removed method call, and I could remove it in my current branch. After deleting all the dead code, I made sure to comment on each removed function in my pull request with the commit number where the consumer was removed (because documentation is rad). Lots of eyes and code review later, our Model was done with her teeth cleaning and felt just a little bit better.
