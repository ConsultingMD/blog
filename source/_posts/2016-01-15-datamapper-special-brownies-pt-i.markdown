---
layout: post
title: "DataMapper: Special Brownies Pt. I"
date: 2016-01-15 16:39:35 -0800
comments: true
categories:
---

Here at Grand Rounds, we use [Integrity](https://github.com/integrity/integrity/tree/master/lib/integrity)
as our CI system. In working with Integrity, I've had the pleasure, adventure... of learning about
[DataMapper](http://datamapper.org/). In this entry, I've decided to write about one of my adventures with
querying in DataMapper.

### Background

Let's say I really like cats, and I am interested in querying the local pet stores for ```Cat```, but my house
can only hold ten cats. No more, and no less. In other words, I also wanted to assert that there were ten
records returned when I added a ```limit: 10``` to my query of the catabase. No problem!

For this example, let us say I have the option to go to two different shops to pick up my cats: Discount Pets,
and Pet Shop Boys. I need to pick up ten, so my I'll just check the ```.count``` on my ```limit: 10``` query.

### DataMapper Query

First, let's read the DataMapper documentation for querying a range:

"If you have guaranteed the order of a set of results, you might choose to only use the first ten results, like this."

```
  @zoos_by_tiger_count = Zoo.all(:limit => 10, :order => [ :tiger_count.desc ])
```

Cool, simple enough!


Let's query Discount Pets!

```
  pry(main)> discount_pets_cats = DiscountPets::Cat.all(breed: 'Norwegian Forest Cat', limit: 10)
  => [#<Cat @id=498 @name="Aristocat" @breed="Norwegian Forest Cat">, #<Cat @id=741 @name="Apple" @breed="Norwegian Forest Cat">,
  #<Cat @id=838 @name="Anteater" @breed="Norwegian Forest Cat">]
```

Looks like there are only three.

```
  pry(main)> discount_pets_cats.count
  => 3
```

Correct! I guess I cannot go to Discout Pets to pick up my ten cats.

Let's query Pet Shop Boys!

```
  pry(main)> pet_shop_boys_cats = PetShopBoys::Cat.all(breed: 'Norwegian Forest Cat', limit: 10)
  => [#<Cat @id=498 @name="Bob" @breed="Norwegian Forest Cat">, #<Cat @id=741 @name="Bernadette" @breed="Norwegian Forest Cat">,
  #<Cat @id=838 @name="Bill" @breed="Norwegian Forest Cat">, #<Cat @id=223 @name="Benson" @breed="Norwegian Forest Cat">,
  #<Cat @id=198 @name="Beavis" @breed="Norwegian Forest Cat">, #<Cat @id=444 @name="Butthead" @breed="Norwegian Forest Cat">,
  #<Cat @id=568 @name="Basil" @breed="Norwegian Forest Cat">, #<Cat @id=782 @name="Brent" @breed="Norwegian Forest Cat">,
  #<Cat @id=366 @name="Beaver" @breed="Norwegian Forest Cat">, #<Cat @id=324 @name="Bruno" @breed="Norwegian Forest Cat">]
```

Neat! If I can count, there are ten records. Right?

```
  pry(main)> pet_shop_boys_cats.count
  => 23
```

Wat.

Five mins of wat-ing, printing out ```pet_shop_boys_cats```, counting to ten. I must be missing something, or
am really dumb.

Trying to examine this, what's the fifth cat?

```
  pry(main)> pet_shop_boys_cats[5]
  => #<Cat @id=444 @name="Butthead" @breed="Norwegian Forest Cat">
```

Makes sense, but since it's telling me there are twenty three cats, what's the twentieth?

```
  pry(main)> pet_shop_boys_cats[20]
  => nil
```

More wat.

```
  pry(main)> PetShopBoys::Cat.all(breed: 'Norwegian Forest Cat').count
  => 23
```

Okay, so that's how they're getting 23... So what is ```pet_shop_boys_cats```?

```
  pry(main)> pet_shop_boys_cats.class
  => DataMapper::Collection
```

So apparently, the ```limit: 10``` does not carry with the query. Great!

Sigh, yes, I'll just have to convert ```pet_shop_boys_cats``` to an array. But should I really have to? Please
tell me that it's a reasonable expectation, let's try it in ActiveRecord!

### ActiveRecord Query

```
  pry(main)> pet_shop_boys_cats = PetShopBoys::Cat.where(breed: 'Norwegian Forest Cat').limit(10)
  => [#<Cat @id=498 @name="Bob" @breed="Norwegian Forest Cat">, #<Cat @id=741 @name="Bernadette" @breed="Norwegian Forest Cat">,
  #<Cat @id=838 @name="Bill" @breed="Norwegian Forest Cat">, #<Cat @id=223 @name="Benson" @breed="Norwegian Forest Cat">,
  #<Cat @id=198 @name="Beavis" @breed="Norwegian Forest Cat">, #<Cat @id=444 @name="Butthead" @breed="Norwegian Forest Cat">,
  #<Cat @id=568 @name="Basil" @breed="Norwegian Forest Cat">, #<Cat @id=782 @name="Brent" @breed="Norwegian Forest Cat">,
  #<Cat @id=366 @name="Beaver" @breed="Norwegian Forest Cat">, #<Cat @id=324 @name="Bruno" @breed="Norwegian Forest Cat">]
```

Cool.

```
  pry(main)> pet_shop_boys_cats.count
  => 10
```

Yep.

```
  pry(main)> pet_shop_boys.cats.class
  => ActiveRecord::Relation
```

Thank you, DataMapper.

### Conclusion

A query with a limit in DataMapper is simply the query with the addition of nilling the elements passed the
given limit, and a ```to_a``` performs a ```.compact```. Weird!
