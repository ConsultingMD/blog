# Blogging

### Installing Octopress
Clone this repo, and assuming you have rvm setup nicely, just bundle install.  It will create a new gemset called `octopress`.

```
git clone git@github.com:ConsultingMD/blog.git
bundle install
```

To run the blog in a non-static, "development" mode, use:

```
rake preview
```

### To Create a Page

```bash
$ rake new_post["title"]
mkdir -p source/_posts
Creating new post: source/_posts/2014-04-11-title.markdown
$ cat source/_posts/2014-04-11-foo.markdown
---
layout: post
title: "title"
date: 2014-04-11 15:40:45 -0700
comments: true
categories:
author:
---
```

The file has a couple of lines of YAML at the top that define the layout, title, etc. Don't forget to add yourself as the author as it might not be generated.

Edit `source/_posts/2014-04-11-title.markdown` with the appropriate markdown to make your blog article AWESOME.


### To push the changes

To generate the new static pages:

```bash
$ rake generate
Generating Site with Jekyll
identical source/stylesheets/screen.css
Configuration from /home/justin/grh/blog/_config.yml
Building site: source -> public
Successfully generated site: source -> public
```

This will have taken all the markup, and generated the html for them.

To push these changes to the live site

```bash
$ aws-environment production
$ rake deploy
## Deploying website via s3cmd
public/atom.xml -> s3://eng.grandroundshealth.com/atom.xml  [1 of 1]
 29878 of 29878   100% in    0s    51.56 kB/s  done
Done. Uploaded 29878 bytes in 4.1 seconds, 38.00 kB/s
OK
```
The changes should now be live.

### Blog images

To embed an image in your blog post, upload it to s3 with public-read access:

```bash
$ aws-environment production
$ aws s3 cp --acl public-read ./my-image.png s3://grh-wiki-pics
```

Then use the following markdown (note the leading '!'):

```markdown
![image description](http://grh-wiki-pics.s3.amazonaws.com/my-image.png)
```

### Shortcut
Add the following post-commit hook into your `.git/hooks/`:

```bash
#!/bin/sh
rake generate
rake deploy
```

Then make it executable `chmod +x .git/hooks/post-commit`

Now on every commit it will update the site.  Use your commits wisely!

## We are awesome.
### We proudly display that we are Octopress
