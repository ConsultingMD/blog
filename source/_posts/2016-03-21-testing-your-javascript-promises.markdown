---
layout: post
title: "Testing your Javascript Promises"
date: 2016-03-21 10:59:50 -0700
comments: true
categories: testing javascript frontend
author: Nancy Foen
---
## Promises
When I started working with JavaScript, I didn't think at all about it being single threaded. As far as I was concerned, a single thread was enough to check field values, update the visibility of my components, and change some of the styles in a page. As my code got more complex, I had to face the limitations of a single thread. For example, I wanted to render new content in a section of the page, but I didn't want to wait for the content to be loaded before updating the fields on the rest of the page.

Since I didn't want to perfom all actions sequentially, but I still wanted to have the ability to chain actions, I started using event listeners and callbacks to handle asynchronous requests. Recently, I began exploring the world of promises. If you haven't used them before, [this](http://www.html5rocks.com/en/tutorials/es6/promises/) is a great resource. With promises, we can do things like this:

```javascript
function updatePage() {
  var loadPromise = new Promise(function(resolver, reject) {
    var link = loadStyleScript();
    if ('onload' in link) {
      link.onload = resolver;
      link.onerror = reject;
    } else {
      resolver();
    }
  });
  // After the stylesheet is loaded, then show the content
  loadPromise.then(showContent);

  // Hide this unrelated content without waiting for the styles
  otherUnrelatedContent.hide();
}
```
Promises are very powerful, but can we write tests for them? Well, we can, but it takes some effort. Now, we can leave promises in our application untested and hope for the best, but since that's not our style, we powered through it and learned some stuff about promise testing.

## Testing
We introduced the concept of JS testing with Konacha in a previous [post](http://eng.grandrounds.com/blog/2015/04/01/writing-js-tests-in-under-ten-minutes/). Now, how do we add asynchronous capabilities?

Testing that a promise works can be done in different ways. If you are returning the promise in your function, you can always do:
```javascript
it ('should succeed!', function (done) {
  return myClass.updatePage().then(function(data) {
    done()
  });
});
```

Notice the *done* argument. This is what your test uses to know it has succeeded. If the *done* method is not invoked, your test will fail after a timeout. If you are not returning the promise, you can use sinon stubs to test that the promise returns successfully. Using the promise example at the beginning:

```javascript
it ('should succeed!', function (done) {
  var showContentStub = sinon.stub(myClass, 'showContent', function() {
    done();
  });
  myClass.updatePage();
});
```

That's neat, but what I struggled testing was the order of execution. How do I make sure that the code in the *then* block actually waits for the loading to be done. Again, using the promise example at the beginning, this is what I did:
```javascript
it ('should wait until the loading is done', function (done) {
  var myValue = 'beforeLoad'
  var linkStub = sinon.stub(myClass, 'loadStyleScript', function() {
    var link = $('<link href="my_app/styles.css" type="text/css" rel="stylesheet"/>');
    setTimeout(function() {
      myValue = 'afterLoad'
      link.trigger('load');
    }, 1000);
    return link[0];
  });

  var showContentStub = sinon.stub(myClass, 'showContent', function() {
    if myValue == 'afterLoad'
      done()
    else
      done('Expected value to be "afterLoad" and it was not)
  });

  myClass.updatePage();
});
```
The timeout adds a buffer to make sure that your loading takes some time, and your show content function is forced to wait. In our example above, passing an argument to *done* fails the example with the given error. Also, make sure that your timeout is not too large, otherwise your tests will fail.

## It works on a browser, but...
Now, we've written some tests to make sure our promise succeeds, and the order of execution is right. Everything seems perfect when we run our konacha tests in our browser. However, if you run it in a console, it fails. Why?!?

Well, it seems like some headless browsers, like PhantomJS, do not support promises yet, at all. So, you can either require [es6-promise](https://github.com/stefanpenner/es6-promise), or you can copy it in a vendor file in your tests, and use that instead. Then, everything should work!

Happy Testing!

## Other testing resources:
[Testing Promises](http://www.sitepoint.com/promises-in-javascript-unit-tests-the-definitive-guide/)
