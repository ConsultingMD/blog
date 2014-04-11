# Blogging
## To Create a Page
```bash
$ rake new_post["title"]
mkdir -p source/_posts
Creating new post: source/_posts/2014-04-11-title.markdown
$ cat source/_posts/2014-04-11-foo.markdown
---
layout: post
title: "foo"
date: 2014-04-11 15:40:45 -0700
comments: true
categories:
author:
---
```
The file has a couple of lines of YAML at the top that define the layout, title, etc.

Edit `source/_posts/2014-04-11-title.markdown` with the appropriate markdown to make your blog article AWESOME


## To push the changes
Add the following post-commit hook into your `.git/hooks/`

```bash
#!/bin/sh
rake generate
rake deploy
```

Then make it executable `chmod +x .git/hooks/post-commit`

Now on every commit it will update the site.  Use your commits wisely!

## Installing Octopress
Clone this repo

Assuming you have rvm setup nicely, just bundle install.  It will create a new gemset called `octopress`.

Then just `bundle install`

To run the blog in a non-static, "development" mode, use `rake preview`.

## We are awesome.
### We proudly display that we are Octopress