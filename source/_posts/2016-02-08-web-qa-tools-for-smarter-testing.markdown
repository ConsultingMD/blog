---
layout: post
title: "Web QA Tools for Smarter Testing"
date: 2016-02-08 18:44:10 -0800
comments: true
categories: qa, tools
author: Ryan Gilbert
---

## Reduce repetitive typing tasks with Dash

[Dash](https://kapeli.com/dash) is a free OSX app that allows you to save text snippets and input them anywhere using custom keywords.

For example, if you have to enter a console command 20 times a day, you can save this command as a snippet in Dash, and title it **`my_command**. If you want to change a variable in this command each time, you can signify a manual entry point by using "__" on either side of the variable: 

![Dash snippet](http://i.imgur.com/RAaq7KF.png)

When you run the snippet by typing **`my_command** in the console, a box will pop up showing the full snippet being entered, with a prompt to enter your variable (in this case, the last name value) manually:

![Entering a dash snippet with variable](http://i.imgur.com/a8ug5pJ.png)

Dash works within ANY application on OSX, I highly recommend it to reduce typing and speed up those repetetive tasks.

[Download Dash](https://kapeli.com/dash)

## Set up a light-weight web server for realistic javascript testing

When testing web code, getting the content on an internet addressable server is necessary to replicate a live web environment. This has been important at Grand Rounds for testing embedded javascript with cross-site requests.

Using the [npm serve package](https://www.npmjs.com/package/serve), it only takes a few minutes to set up.

1. Install the web server:

```
npm install serve
```

2. Edit your host file (/etc/hosts) to include a hostname other than localhost:

```
127.0.0.1       www.l.grandrounds.com
```
**Why add a new host name?**
When your browser connects to a hostname other than localhost, it treats it as an external internet addressable server. This is important for replicating a live environment.

3. Create a directory containing the pages you want to test:

```
mkdir Pages
```

4. In terminal, go to that directory, and serve it:
```
cd Pages
serve
```
If successful, you should see:

serving /Users/Pages on port 3000

5.  In a web browser, open that directory from the server: http://www.l.grandrounds.com:3000/

In your browser, you should see the directory being served, and be able to view your pages from here in a live environment.

Happy Testing!