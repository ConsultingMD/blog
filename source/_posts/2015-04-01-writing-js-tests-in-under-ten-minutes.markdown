---
layout: post
title: "Writing JS/CoffeeScript Tests in Ten Minutes"
date: 2015-04-01 19:02:59 -0700
comments: true
categories: testing
---
Writing javascript tests are actually quite simple with the tools we have at our disposal. The testing frameworks we use permit us to create tests very similar to those rspec tests we have grown accustomed to. Many of us feel lost when having to write konacha tests, and my hope is that this quick tutorial will serve as a good intro into the world of javascript testing.

##Getting Started

1. Add the konacha gem as a dependency in our Gemfile
2. Download sinon.js and add it to our javascript vendor assets(will come in handy when we want to stub the server or travel in time:))
3. Create a spec/javascripts directory

##Writing Tests(the good part)

Let us start by creating a helper file named spec/javascripts/spec_helpers.js.coffee which we will use to require important assets for our tests.
```
#= require jquery
#= require chai-jquery
#= require sinon
#= require_tree ./templates
```

We will now create a coffeescript file where we will have a Test object with a few functions: app/assets/javascripts/test.js.coffee
```
Test =
  #Tests if word_a is a prefix of word_b, ex: prefix('anti', 'antialiasing')
  prefix: (word_a, word_b) -> word_b.slice(0, word_a.length) == word_a;

  #simple ajax request
  request: (callback)->
    url = "/foo"

    $.ajax
      url: url
      type: 'GET'
      dataType: 'script'
      success: (data) ->
        callback(null, data)

(exports ? this).Test = Test
```

Next, we will add a template file to simulate javascript code that modifies HTML in our views: spec/javascripts/templates/test.jst.skim
```
input id='foo'

```


And finally, we add our test file: spec/javascripts/test_spec.js.coffee
```
#= require spec_helpers
#= require test

describe 'Test#prefix', ->
  describe 'a prefix', ->
    it 'returns true', ->
      expect(Test.prefix('anti', 'antialiasing')).to.be.true
  describe 'not a prefix', ->
    it 'returns false', ->
      expect(Test.prefix('nti', 'antialiasing')).to.be.false

describe 'Test#request', ->
  describe 'success', ->
    server = null
    spy = null
    fake_response = ->
      server.requests[0].respond(200,{},'')

    beforeEach ->
      #stubbing the server
      server = sinon.fakeServer.create();

    #How to test that a function gets called
    it 'calls our callback function', ->
      spy = sinon.spy()
      Test.request(spy)
      fake_response()
      expect(spy.calledOnce).to.be.true

    #Input in the template
    describe 'the foo input', ->
      clock = null
      time = 10000 #10 seconds

      callback = (err, data)-> setTimeout (-> $('#foo').addClass('bar')), time

      beforeEach ->
        #loading the template
        $('body').html(JST['templates/test'])
        #time is now initialized at 0
        clock = sinon.useFakeTimers()
        Test.request(callback)
        fake_response()

      afterEach ->
        clock.restore()

      describe 'before timeout', ->
        it 'does not have the bar class', ->
          expect($('#foo')).to.not.have.class('bar')

      describe 'after timeout', ->
        beforeEach ->
          clock.tick(time+1000) #timeout time + 1s
        it 'has the bar class', ->
          expect($('#foo')).to.have.class('bar')
```
Those who have written tests in rspec will see the resemblance almost immediately. This set of tests might not be the most beautiful JS/Coffee tests ever written, but they serve as a complement to the konacha readme and give a good overview on how to structure and write basic tests.

##Conclusion

As we start creating applications with more logic on the front end, it will become increasingly important that we structure our JS/Coffee code in a modular way to facilitate testing and that we become comfortable doing so. Writing JS tests is a nightmare for many of us, but it does not have to be that way.

###External Resources
- http://mochajs.org/
- http://chaijs.com/
- http://sinonjs.org/
