---
layout: post
title: "Rails Active Records: assigning a non-boolean to a boolean"
date: 2016-11-28 16:57:10 -0800
comments: true
categories: ActiveRecords, IntegerConversion
author: "Subhendu Aich <subhendu.aich@grandrounds.com>"
---
We'll discuss how Active Records handle data assignment to a numeric attribute. I observed this 'odd' behavior while working in my last epic. 


We'll work with boolean attribute here because Mysql treats boolean data as numeric. Mysql doesn’t have inherent data type called ‘boolean’. When create a column of type boolean internally stores the binary state in a 'tinyint' (1 byte datatype which holds integer values in the range -128 to 127). TRUE , FALSE are simple constants which evaluate to 1 & 0.


Now let’s imagine Active Record trying to work with a boolean column in mysql. What happens when we assign string data to a boolean attribute in AR. Active Record will try and coerce the data set in the attribute to a number (because boolean is numeric in mysql). Great!!! 
How does it convert string to int ? 


I know 2 different ways this can be achieved in ruby.

```
pry(main)> Integer('23')
=> 23
[4] pry(main)> '23'.to_i
=> 23
```

Interesting to see the behavior of the above methods when try to cast a non integer to integer. 

```
pry(main)> 'asdf'.to_i
=> 0
[2] pry(main)> Integer('asdf')
ArgumentError: invalid value for Integer(): "asdf"
from (pry):2:in `Integer'
```

The #Integer method complains whereas the use of #to_i results in 0. Unfortunately Active Record uses #to_i to set a boolean attribute and results in FALSE for any non boolean assignment.  :-(


Here's what happened :- 

```
pry(main)> ds = DataSource.find(5)
  DataSource Load (0.3ms)  SELECT `data_sources`.* FROM `data_sources` WHERE `data_sources`.`id` = 5 LIMIT 1
=> #<DataSource id: 5, auto_approval_enabled: true>
[2] pry(main)> ds.auto_approval_enabled
=> true
[3] pry(main)> ds.auto_approval_enabled = 'asdf'
=> "asdf"
[4] pry(main)> ds.save!
=> true
[5] pry(main)> ds.reload.auto_approval_enabled
=> false
```
The world is bad. Not really ..

We only observe this behavior in Active Record 3. With Active Record 4 it throws the much needed warning for non-boolean to boolean assignment.  

```
DEPRECATION WARNING: You attempted to assign a value which is not explicitly `true` or `false` 
("asdf") to a boolean column. Currently this value casts to `false`. This will change to match Ruby's 
semantics, and will cast to `true` in Rails 5. If you would like to maintain the current behavior, you 
should explicitly handle the values you would like cast to `false`.
```
Much better, right ??
