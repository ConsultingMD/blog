---
layout: post
title: "Installing Zulip Linux Client from Source"
date: 2015-10-20 18:13:50 -0700
comments: true
author: Alex Bird
categories: tools, general-stuff
---

We started using the newly-released [Zulip](https://zulip.org/) at Grand Rounds
recently, as it gives us full HIPAA-complience and in my opinion a great chat
experience. I have been using their Linux client, which I prefer over the
browser-based client due to it being an Alt-Tab away as opposed
to buried in the middle of a bunch of browser tabs.

The only problem I've been experiencing is that code blocks are not formatted
in a monospace font. Just ghastly. Fortunately, they have fixed this in master,
but the fix hasn't made it into their provided Ubuntu package yet. Why not run
from source?

```
# install some build dependencies
sudo apt-get install libqt5svg5-dev libqt5webkit5-dev qtmultimedia5-dev

# get the source
git clone git@github.com:zulip/zulip-desktop.git
cd zulip-desktop/
mkdir build
cd build
cmake -DBUILD_WITH_QT5=On ..
make

# run it
./zulip --site https://my.private.zulip.foo
```

I prefer reading code in a fixed-width font.
