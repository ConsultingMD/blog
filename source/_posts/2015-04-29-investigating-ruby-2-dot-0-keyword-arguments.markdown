---
layout: post
title: "Problems with Ruby 2 Keyword Arguments"
date: 2015-04-29 17:23:52 -0700
comments: true
categories:
author: Ozzie Gooen <ozzie@grandrounds.com>
---

Recently I started using [keyword arguments](https://robots.thoughtbot.com/ruby-2-keyword-arguments) in my ruby work.  They were fun at first, but I eventually became frustrated with their added complexity and inconsistencies with ordered and hash parameters.

### 1. Keyword arguments don’t convert into hashes

Hash arguments are really flexible.  They can be modified and passed into other functions, as shown below.

``` ruby
def make_sandwich(ingredients)
  prepare_spread(incredients.slice(:fruit, :sugar))
  run!(ingredients)
end
```

Keyword arguments can’t do this.  Instead, they need to be explicitly stated.

``` ruby
def make_sandwich(fruit:, sugar:, peanuts:, salt:, bread:, knife:)
  prepare_spread(fruit: fruit, sugar: sugar)
  run!(fruit: fruit, sugar: sugar, peanuts: peanuts, knife: knife)
end
```

One solution to this is converting the named argument inputs into a hash, but this is surprisingly tricky.  The best way to do this was suggested by [Dennis on Stackoverflow](http://stackoverflow.com/questions/22026694/ruby-keyword-arugments-can-you-treat-all-of-the-keyword-arguments-as-a-hash).

``` ruby
# Method 1
def call(a:, b:, c:)
  opts = method(__method__).parameters.map(&:last).map { |p| [p, eval(p.to_s)] }.to_h
  foo(opts)
end

```

Hashes can easily be converted to keyword arguments, so it’s strange that it’s so difficult to convert keyword arguments into hashes.

### 2. Keyword argument calls don’t use variable names

Keyword arguments can be quite non-dry from the context of the calling function.

```
plants = 3
animals = 38
humans = 83
computers = 39
#later
computers = get_updated_computers(computers)

call(plants: plants, animals: animals, humans: humans, computers: computers)
```

We could use a hash to dry this up.

```
things = {
  plants: 3,
  animals: 38,
  humans: 83,
  computers: 39
}

#later
things[:computers] = get_updated_computers(things[:computers])

call(**things)
```

Of course, using a hash for this destroys some of the point of using named parameters.  We need to make a **things** hash and the naming is longer than ideal.

Compare this to the following theoretical example.

```
plants = 3
animals = 38
humans = 83
computers = 39
#later
computers = get_updated_computers(computers)

call(plants:, animals:, humans:, computers:)
```

The downside to this, of course, is that it would require one to use the keywords `animals`, `humans`, and `computers` as their variable names before these calls.  However, this may be a really good thing.  It would enforce more similar naming conventions throughout an application.

### 3. Implicit hash parameters are easy to get confused with keyword arguments

What do you expect this following code to produce?

```
def foo(a, b:3)
  puts “a is #{a}”
  puts “b is #{b}”
end

foo(b:89)
```

The answer is:
```
a is {:b=>89}
b is 3
```

Because `b:89` was the first argument, ruby assumes that it represents the beginning of a hash parameter, so will take all key:value arguments and apply them to `a` as a hash. 

However, if `a` has a default value it acts as would be originally expected.

```
def foo(a=0, b:3)
  puts “a is #{a}”
  puts “b is #{b}”
end

foo(b:89)

# a is 0
# b is 89
```

### 4. It is difficult to make keyword arguments drop-in replacements for positional arguments
As one expands their use of keyword arguments, opportunities should arise to validate outputs before runtime.  However, most methods don’t yet feature keyword arguments.

```
def make_sandwich(bread=3, cheese)
end
```

What if we want a method to accept either positional arguments or named arguments?  A user could then call `make_sandwich(bread, cheese)` or `make_sandwich(bread:bread, cheese:cheese)`?  This could be because `make_sandwich` has been around before Ruby 2.0, but we want to use keyword arguments in the future of the codebase.

```
def make_sandwich(bread, cheese=nil)
  if bread.is_a? Hash
    cheese = bread[:cheese] || raise ArgumentError
    bread = bread[:bread] || 3
  end
  #more code
end
```
Or with delegation

```
  def make_sandwich(bread, cheese=nil)
    if bread.is_a? Hash
      keyword_arg_make_sandwich(bread)
    else
      keyword_arg_make_sandwich(bread: bread, cheese: cheese)
    end

  def keyword_arg_make_sandwich(bread:3, cheese:)
    #more code
  end
```

These both are terrible.  Perhaps there are more elegant solution, but there doesn’t seem to be something obvious, especially something standardized.

## What does the future hold?

Keyword arguments are interesting, but their relationship with hashes and positional arguments is messy and a bit confusing.  This may get cleaned up, but perhaps a better solution would be to do a complete rethink of parameters.

Ruby 3.0 may introduce [static typing](https://www.omniref.com/blog/blog/2014/11/17/matz-at-rubyconf-2014-will-ruby-3-dot-0-be-statically-typed/), with syntax like the following:

```
def connect(r -> Stream, c -> Client) ->  Fiber
```

Hopefully this will not end up in four different kinds of arguments, but rather will be done in a large parameter consolidation  If so, keyword arguments may only exist temporarily in the history of Ruby.  They may be handy to use briefly in new ruby projects that support them, but I would advise against doing a large refactor to implement them.  You’ll likely have to introduce more biolerplate than you expect to.
