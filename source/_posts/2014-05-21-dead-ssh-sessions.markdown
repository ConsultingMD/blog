---
layout: post
title: "Dead SSH Sessions"
date: 2014-05-21 14:07:36 -0700
comments: true
categories:
author: Ken Berland
---

Sometimes you're using `ssh` on a wired connection and you move to an unwired connection, for example.  This switches your interface and `ssh` gets unhappy about that; you lose your connection.  Instead of closing the terminal and losing the history as well, try hitting the key combination of return, then `~`, then `.`

This should close the connection and free-up the shell.
