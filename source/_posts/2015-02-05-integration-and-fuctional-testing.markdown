---
layout: post
title: "RSpec and large test files"
date: 2015-02-06 00:20:12 -0800
comments: true
categories: testing
author: Bojil Velinov <bojil@grandroundshealth.com>
---

#### The Idea #
I had the initial idea, this week, to write about the complexity of testing a modern commercial application, and the importance of integration and feature testing. I was going to have a rhetorical view of the software as a whole system, it's modularity but at the same time unity. But instead, decided to write a much more practical blog, a tip that helps you divide and conquer large pieces of spec/test code and it's execution.

#### In Practice #
Often, when writing a spec for a particular feature or almost any other Rails spec authors have the tendency to get carried out - have a large amounts of test scenarios all packed neatly in one large file. The end result of that could be file with more then 3K lines of code, plus their accompanying helper definition/methods. This detailed, but cumbersome file becomes hard to digest, which in it’s end makes it hard to read, modify and improve, i.e. harder to make it DRY-er. Very often the reason to have such large files lies in the connectives of the test, or the fact that they are all for the same model, controller, etc.

So how do we improve their readability while keeping their logical collectiveness?

#### The Tip #
By using symbols as an additional metadata to your test.
This way you can split the large file into smaller chunks, but still execute it as one, while you're focused on a it as problem/test.

Filter the examples using the `config.filter_run` option.

In your `spec_helper.rb` config part specify:
```
RSpec.configure do |c|
  c.filter_run :my_epic_example
end
```

While in your spec files have:
```
RSpec.describe "something" do
  it "does one thing" do
  end

  it "does another thing", :my_epic_example do
  end
end
```
```
RSpec.describe "something new”, :my_epic_example do
  it "does second thing" do
  end

  it "does third thing" do
  end
end
```
When we run this the output should contain *“does another thing”*, *“does second thing”*, and *“does third thing”*.

Via command line you can user the -t option.
As such:
```
rspec . —-tag my_epic_example
```

If you would like to exclude these examples you can simply run `rspec . —-tag ~my_epic_example`

An additional benefit of organizing your test files into a smaller ones would be their faster execution with `parallel_spec`, i.e. their distribution will be more even, and you wouldn't need to wait for large files to complete and free resources.

Cheers!