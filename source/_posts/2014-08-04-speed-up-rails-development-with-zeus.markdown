---
layout: post
title: "Speed Up Rails Development With Zeus"
date: 2014-08-04 16:37:11 -0700
comments: true
categories: 
---

####Zeus

One of the first things that made me love Ruby and (Rails by association), and move away from Java was speed. I don’t mean performance or load times. I mean the velocity at which I could build and deliver quality software. I hate waiting. Waiting for pages to load, waiting for code to compile, waiting for tests to run… waiting is one of the most annoying things about software development. 

In the beginning Rails was fast.The time it took to develop web applications decreased significantly. But as Rails got bigger and more complex, and the things I built with Ruby and Rails got more sophisticated, I noticed things start to slow down. Boot times increased. Running tests no longer took seconds, but minutes. I (and many others in the Rails community) began to look for tools to help speed up development. One tool in particular has become a favorite of mine and among my comrades here at Grand Rounds. 

**Zeus!** Zeus is a Ruby gem that helps you to significantly speed up development time in Rails. Zeus preloads your Rails app so that your normal development tasks such as console, server, generate, and specs/tests take less than one second.

####Installing

All you need is *Ruby* and *Rails* installed in order to use it. Make sure you are using *Ruby 1.9.3* or later. With Rails you should be on version 3.x or later.  With the prerequesits, you just need to install Zeus itself, which can be installed like any other Ruby Gem:

```
gem install zeus
```

After you’ve installed it, you can use the zeus command to start it up:

```
zeus start
```

You’ll see zeus come up and it’ll list the available commands:

```
[ready] [crashed] [running] [connecting] [waiting]
boot
└── default_bundle
    ├── development_environment
    │   └── prerake
    └── test_environment
        └── test_helper

Available Commands: [waiting] [crashed] [ready]
zeus rake
zeus runner (alias: r)
zeus console (alias: c)
zeus server (alias: s)
zeus generate (alias: g)
zeus destroy (alias: d)
zeus dbconsole
zeus test (alias: rspec, testrb)
```

If you find that you don’t use one or more of the commands, you can remove them by editing the zeus.json config file.

```
{
  "command": "ruby -rubygems -r./custom_plan -eZeus.go",

  "plan": {
    "boot": {
      "default_bundle": {
        "development_environment": {
          "prerake": {"rake": []},
          "runner": ["r"],
          "console": ["c"],
          "server": ["s"],
          "generate": ["g"],
          "destroy": ["d"],
          "dbconsole": []
        },
        "test_environment": {
          "cucumber_environment": {"cucumber": []},
          "test_helper": {"test": ["rspec", "testrb"]}
        }
      }
    }
  }
}
```

####Using Zeus

Now that you got Zeus installed and running, it's time to experience the awesome! Nearly all of the commands you normally run in Rails, you can now run in zeus. Note, that you no longer need to prefix the commands with *bundle exec* when using zeus. Here's a list of all the commands available:

```
zeus console
zeus server
zeus rspec
zeus test
zeus cucumber
zeus dbconsole
zeus generate
zeus destroy
zeus rake
zeus runner
```



When you start up your server with zeus, you'll notice that it seems to come up instantly. That's because zeus has pre-loaded the environment and the Rails app.

```
rconda@localhost: ~/Projects/mysampleapp $ zeus s
=> Booting Thin
=> Rails 3.2.16 application starting in development on http://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
>> Thin web server (v1.5.1 codename Straight Razor)
>> Maximum connections set to 1024
>> Listening on 0.0.0.0:3000, CTRL+C to stop
```

You'll see the same kind of speed when you're starting up Rails console. You get the irb prompt in less than a second:

```
rconda@localhost: ~/Projects/mysampleapp $ zeus c
Loading development environment (Rails 3.2.16)
1.9.3-p484 :001 > 
```

####Running Specs

Beyond the normal server commands, zeus also speeds up running tests, or in my case specs.

```
zeus rspec spec/models/user_spec.rb
```

You'll notice that there is virtually no load time. The specs start running right away.

You may also want to remove the *auto* commands from spec_helper.rb They are known to cause issues when running with zeus:

```
 - require 'rspec/autotest'
 - require 'rspec/autorun'
```
