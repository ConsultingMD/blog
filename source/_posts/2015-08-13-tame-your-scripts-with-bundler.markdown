---
layout: post
title: "Tame your scripts with Bundler"
date: 2015-08-13 00:00:00 -0700
comments: true
author: Josh Strater
categories:
---

At Grand Rounds we write a lot of scripts to automate our development process and link together the services we use. For example, we have a tool that automatically names git branches based on their corresponding stories in [Pivotal Tracker](http://www.pivotaltracker.com/). We also have scripts to infer what features are included in a release based on git commit logs.

### Script dependencies are annoying

Since we're primarily a Ruby shop, it's often convenient to use Ruby for these scripts. Usually they start out as a simple executable file with `#!/usr/bin/env ruby` at the top, followed by a few `require` statements. Put the file somewhere in your `$PATH` and it's ready to go.

When you start requiring gems for API clients, argument parsing, and more, though, dependency management becomes a problem. Simply requiring everything you need at the top of the file leaves users with the chore of manually installing a bunch of gems and keeping them up to date.

### Bundler to the rescue

That's the kind of problem that RubyGems and [Bundler](http://bundler.io/) were built to solve. If you write Rails applications, you're probably familiar with the most common use case for Bundler: a project will include a `Gemfile` where gem dependencies are laid out, and Bundler provides commands that make it easy to install and maintain them. Fortunately, it's also versatile enough to help maintain your scripts' dependencies by packaging them as gems. The built-in generator makes the process easy.

What follows is a quick & dirty guide to turning a standalone script into a gem. There's a lot more to gems than I'll cover here, but if you just want a way to get a script's dependencies under control, this gets the job done.

# An example

Here's a script I wrote that uses the Tracker API to list the ten biggest stories in a project.

```ruby
#!/usr/bin/env ruby

require 'tracker_api'
require 'colorize'

# If you run this script, don't forget to set these environment variables!
TRACKER_PROJECT_ID = ENV['TRACKER_PROJECT_ID']
TRACKER_TOKEN = ENV['TRACKER_TOKEN']

client = TrackerApi::Client.new(token: TRACKER_TOKEN)

stories = client.project(TRACKER_PROJECT_ID).stories(
  filter: 'includedone:false type:feature -estimate:-1',
  fields: ':default,estimate')
ten_biggest_stories = stories.sort_by(&:estimate).reverse.take(10)

ten_biggest_stories.each do |story|
  color = :red
  color = :yellow if story.estimate < 4
  color = :green if story.estimate < 3

  puts "#{story.estimate.to_i} - #{story.name}".colorize(color)
end
```

### Generating a gem

To turn this into a gem, I'll use the [`bundle gem`](http://bundler.io/bundle_gem.html) command to generate a skeleton for the project.

```text
$ bundle gem --edit --bin bigstories
Creating gem 'bigstories'...
      create  bigstories/Gemfile
      create  bigstories/.gitignore
      create  bigstories/lib/bigstories.rb
      create  bigstories/lib/bigstories/version.rb
      create  bigstories/bigstories.gemspec
      create  bigstories/Rakefile
      create  bigstories/README.md
      create  bigstories/bin/console
      create  bigstories/bin/setup
      create  bigstories/.travis.yml
      create  bigstories/exe/bigstories
Initializing git repo in /Users/jstrater/projects/bigstories
         run  mvim --nofork "/Users/jstrater/projects/bigstories/bigstories.gemspec" from "."
```

Bundler just did most of the work for me, and it finished by opening `bigstories.gemspec` in my editor so that I can fill in the [gem specification](http://guides.rubygems.org/specification-reference/). I'll fill in the lines marked with `TODO`, add my dependencies, and save the file.

```ruby
Gem::Specification.new do |spec|
  [...]
  spec.summary       = %q{Lists the top 10 biggest features}

  spec.add_dependency "tracker_api", "~> 0.2"
  spec.add_dependency "colorize", "~> 0.7"
  [...]
end
```

Since I added the `--bin` flag to `bundler gem`, Bundler automatically created `exe/bigstories` for me. I'll copy & paste my Tracker script into it.

### Done already?

The gem is ready to install, and all I had to do was

1. run `bundle gem`,
2. fill in the gemspec, and
3. add my script to `exe/`.

To install the gem locally, I'll run

```text
$ rake install
bigstories 0.1.0 built to pkg/bigstories-0.1.0.gem.
bigstories (0.1.0) installed.
```

That's it. My `bigstories` gem is installed and ready to go:

```text
$ bigstories
3 - Apply styling to all shopper facing parts of the site, based on assets from designer
3 - Shopper should be able to search for product
3 - When shopper submits order, authorize total product amount from payment gateway
2 - Signed in shopper should be able to post product reviews
2 - Shopper should be able to view contents of shopping cart
2 - Shopper should be able to check status of order by entering name and order number
2 - Shopper should be able to sign up for an account with email address
2 - Signed in shopper should be able to save credit card and address information used in checkout
1 - Admin can review all order questions and send responses to shoppers
1 - When checking out, shopper should have the option to sign in to their account
```

Success!
