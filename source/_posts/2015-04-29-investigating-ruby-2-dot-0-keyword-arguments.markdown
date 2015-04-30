---
layout: post
title: "Ruby 2 Keyword Arguments: Interesting, but Incomplete"
date: 2015-04-29 17:23:52 -0700
comments: true
categories:
author: Ozzie Gooen <ozzie@grandrounds.com>
---


What are the benefits of Keyword arguments?
1. Ordering
2. Validation
3. Sugar for default values
4. Metaprogramming

Recently I worked on a ruby project that involved a lot of passing option hashes around between objects.  This turned into a lot of grunt work to understand exactly which arguments each method needed, so I began work on some meta-programming and interface declaration, like so:
```
class MakeSandwich

  INTERFACE = {
    options: [:sandwich_style]
    inputs: [:bread, :meat, :lettuce, :spread]
    outputs: [:sandwhich]
  }

  def initialize(options)
    @options = options.slice(INTERFACE[:options])
  end

  def call(inputs)
    @inputs = inputs.slice(INTERFACE[:inputs])
    @outputs = {}
    do_stuff_here(inputs)
    return @outputs
  end
end
```

This was inspired by static typing and also by React Components, which encourage the declaration of state on the top of the class for validation.

A coworker suggested using the new Ruby Keyword Argument feature to do this for me, and I decided to give it a go.  The above class became something like:

```
class MakeSandwich

  def initialize(sandwich_style:)
  end

  def call(bread:, meat:, :lettuce:, :spread)
    @outputs = {}
    do_stuff_here(bread, meat, lettuce, spread)
    return outputs
  end
end
```

This was obviously shorter, and came with built-in errors for parameter mismatching, which was awesome.  However, it also came with several limitations which surprised me.

## Keyword arguments don’t convert into hashes

Hashes convert into keyword arguments, but keyword arguments won’t convert into hashes.  In the above example, I was able to pass the inputs to `call` straight into a subcommand, `do_stuff_here`.  I find it is fairly natural to easily take all or a subset of the inputs to a function and send them to a different function.  The keyword argument approach is longer and more brittle to changes in variable names.

There are a few ways of getting around this, as suggested by [Dennis on Stackoverflow](http://stackoverflow.com/questions/22026694/ruby-keyword-arugments-can-you-treat-all-of-the-keyword-arguments-as-a-hash). However, they aren’t elegant.  The most realistic is this long line below.  This can be converted into a method, but it should be part of the language.  It’s too fundamental.

```
# Method 1
def call(a:, b:, c:)
  opts = method(__method__).parameters.map(&:last).map { |p| [p, eval(p.to_s)] }.to_h
end

```
## Keyword argument outputs don’t use variable names

Variables can be called directly when keyword arguments are used as inputs, as so:
```
def call(a:, b:, c:)
  puts a
  puts b
  puts c
end
```

However, this interpolation does not work with outputs.

```
animals = 38
humans = 83
computers = 39
#later
computers = get_updated_computers(computers)
call(animals: animals, humans: humans, computers: computers)
```  
This is not dry and can easily become quite long.  Compare this to the following theoretical example.

```
call(animals:, humans:, computers:)
```

The downside to this, of course, is that it would require one to use the keywords `animals`, `humans`, and `computers` as their variable names before these calls.  However, this may be a really good thing.  It would enforce more similar naming conventions throughout an application.

Fortunately hashes can be sent to methods that require keyword arguments.  This is essential.

```
options = { animals: 38, humans: 83, computers: 39 }
#later
options[:computers] = get_updated_computers(options[:computers])
call(options)
```  

This gets around the issue of having to restate each variable name in the ```call``` method, but the hash takes up extra line space (```options[:computers]``` vs. ```computers```)

## Implicit hash parameters are easy to get confused with keyword arguments

What do you expect this following code to produce?

```
def fo(a, b:3)
  puts “a is #{a}
  puts “b is #{b}
end

fo(b:89)
```

The answer is:
```
a is {:b=>89}
b is 3
```

Because `b:89` was the first argument, ruby assumes that it represents the beginning of a hash parameter, so will take all key:value arguments and apply them to ```a``` as a hash.  Unless these are put inside of curly braces, it appears impossible to declare the keyword argument value of `b`.

Hash parameters were useful, but become quite confusing with keyword arguments.

## It is difficult to make keyword arguments drop-in replacements for positional arguments
As one expands their use of keyword argument, opportunities should arise to validate outputs before runtime.  However, most methods don’t yet feature keyword arguments.

```
def make_sandwich(bread=3, cheese)
end
```

What if we want a method to accept either positional arguments or named arguments?  A user could then call ```make_sandwich(bread, cheese)``` or ```make_sandwich(bread:bread, cheese:cheese)```?  This could be because ```make_sandwhich``` has been around before Ruby 2.0, but we want to use keyword arguments in the future of the codebase.

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
      keyword_arg_make_sandwhich(bread: bread, cheese: cheese)
    end

  def keyword_arg_make_sandwich(bread:3, cheese:)
    #more code
  end
````

## Conclusion
Keyword arguments are really interesting, but their relationship to hashes and positional arguments is messy and a bit confusing.  I kind of wish they were implemented as a kind of hash rather than a new argument type.
