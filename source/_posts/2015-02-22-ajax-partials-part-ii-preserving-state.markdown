---
layout: post
title: "AJAX Partials Part II - Preserving State"
date: 2015-02-22 11:21:20 -0800
comments: true
categories: frontend ajax parjax jquery
author: Brett Suwyn <brett@grandroundshealth.com>
---
In [Part 1](/blog/2014/06/03/rolling-your-own-ajax-partials-with-jquery-rails/)
we created a simple jQuery plugin that progressively enhances anchor
elements within a HTML document to request and render page content inline using AJAX.

In this post, we will ready our plugin for prime time by adding statefulness.

The problem we identified at the end of
[Part 1](/blog/2014/06/03/rolling-your-own-ajax-partials-with-jquery-rails/),
was that if a user refreshed their web browser they would lose the content
they had previously navigated to.   While our plugin "Parjax" enhanced the
user's experience by decreasing perceived request and rendering times, it
also degraded the user's experience.   After navigating through a Parjax link there was no way
for the user retain that view of the page as they were seeing it.
Parjax did not support refreshes of the web page, bookmarking, or sharing.   Yes, we broke the Internets.

##### What's the correct solution?

Knowing HTTP by [definition](https://tools.ietf.org/html/rfc7230) is a stateless
protocol (every request to the server must contain all the information required
to fulfill that request) we know we must either use the document URL or cookies to
preserve the state of the page.

So is the correct solution a cookie or something in the document URL?   The problem with
using a cookie is that it is persisted with the browser.   So unless we want to
hand our computer to everyone we want to share our page with, we should use the
document URL.   Further, we should use the anchor property of a URL to store this
information, as it was intended to serve exactly the purpose we need it for,
linking [document fragments](http://www.w3.org/TR/html401/intro/intro.html#fragment-uri).

Well, now that we know how we want to do it, let's get to the fun stuff, coding!

###Parjax v0.01

Here is where we left off.   A jQuery plugin that binds itself to any anchor tag within
the element we call `parjax()` on:

```
// jquery.parjax.js
(function($) {

  $.fn.parjax = function() {
    return this.each(function() {
      $container = $(this);

      $container.on('click', 'a', function(event) {

        // load the link via ajax and set the response into our Parjax element
        $container.load($(this).prop('href') + ' [data-parjax-container] *');

        // don't let the browser do its thing
        event.preventDefault(true);

      });

    });
  }

})(jQuery);
```

Knowing that we would like to persist the link the user clicks on to the document
location the logical place to put our logic is after we load the content into the container.

Luckily for us, this is very easy to do.   Within the click handler the function
context `this` is conviently the anchor element the user had clicked on.
There are a number of properties the
[anchor element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a) exposes,
and what we are interested in is the href property value.   We can access this via
the DOM (no need for jQuery) via `this.href`.   But while our anchor element's href
attribute value may be relative, the anchor's `href` property returns the full URL
for the resource. e.g. "http://localhost:9000/part2/index".

#### That Spidey sense

Immediately our little Spidey programmer sense should go off.   There are two items
to note about `this.href`:

1. The scheme and host are redundant with the document's scheme and host.
2. Handle the case when an external link is clicked.

Spidey *is* trying to tell us something, but let's revisit those after we get to
where we are currently going.

### Persisting the link URL

To persist the link URL, we'll use the `document.location.anchor` property of the
DOM and set that to the anchor element's href.   Our modified click handler would look like:

```

      $container.on('click', 'a', function(event) {

        // load the link via ajax and set the response into our Parjax element
        $container.load($(this).prop('href') + ' [data-parjax-container] *');

        // persist the link that was clicked
        document.location.hash = this.href;

        // don't let the browser do its thing
        event.preventDefault(true);

      });

```

Running our example again, we see it now works as we'd like.  Clicking any of our
Parjax'd links we see the browser address bar update with the href set into the document
fragment identifier.  In my example, it looks like
`http://localhost:9000/part2/index#http://localhost:9000/part2/partial1`.
Pretty dang ugly if you ask me.

### Cleaning up the document identifier

So lets clean up #1 on our Spidey list above to restore sanity.
This logic is straightforward, lets create a `baseURL` property that concatenates our
document's scheme and host
`baseUrl = document.location.protocol + '//' + document.location.host;` that we then can use
to strip out of the anchor element's URL value `this.href.replace(baseUrl, '')`.

```

      var baseUrl = document.location.protocol + '//' + document.location.host;

      $container.on('click', 'a', function(event) {

        // load the link via ajax and set the response into our Parjax element
        $container.load($(this).prop('href') + ' [data-parjax-container] *');

        // persist the link that was clicked
        document.location.hash = this.href.replace(baseUrl, '');

        // don't let the browser do its thing
        event.preventDefault(true);

      });

```

Running our example again we see our new Parjax'd URL updates to
`http://localhost:9000/part2/index#/part2/partial1`.   Ahh that's better.   Let's
cross that from the Spidey list and add a new item.

1. ~~The scheme and host are redundant with the document's scheme and host.~~
2. Handle the case when an external link is clicked.

### Handling external links

Moving onto Spidey item #2, we just need to add a simple clause to only enhance
anchor links that reference our pages since we don't have a clear plan on how to
handle external links through Parjax yet.   Since we follow the progressive
enhancement mantra, we'll let those links fall to the default browser behavior.

```

      var baseUrl = document.location.protocol + '//' + document.location.host;

      $container.on('click', 'a', function(event) {

        // only use Parjax when we are requesting our own pages
        if (this.host === document.location.host) {

          // load the link via ajax and set the response into our Parjax element
          $container.load($(this).prop('href') + ' [data-parjax-container] *');

          // persist the link that was clicked
          document.location.hash = this.href.replace(baseUrl, '');

          // don't let the browser do its thing
          event.preventDefault(true);

        }

      });


```

1. ~~The scheme and host are redundant with the document's scheme and host.~~
2. ~~Handle the case when an external link is clicked.~~

### Loading the persisted state
We now have a document URL that represents our page as we are viewing it.
All that remains is handling the case where the page is loaded with a document
fragment identifier already present in the URL.  For example someone entering
`http://localhost:9000/part2/index#/part2/partial1` directly into their browser
address bar should see the content from our view `part2/partial1.erb` rather
than `part2/index.erb`

What we want to do is detect the presence of `document.location.anchor` value and if it exists,
load the URL into our Parjax element.   We can use the same `load()` statement we
used in the click handler.

```

      var baseUrl = document.location.protocol + '//' + document.location.host;

      $container.on('click', 'a', function(event) {

        // only use Parjax when we are requesting our own pages
        if (this.host === document.location.host) {

          // load the link via ajax and set the response into our Parjax element
          $container.load($(this).prop('href') + ' [data-parjax-container] *');

          // persist the link that was clicked
          document.location.hash = this.href.replace(baseUrl, '');

          // don't let the browser do its thing
          event.preventDefault(true);

        }

      });

      // detect a deep link
      if (document.location.hash.length > 0) {

        // remove the # from the fragment identifier and make an ajax request to load the content
        $container.load(document.location.hash.substr(1) + ' [data-parjax-container] *');

      }

```

Running our example, we're quite please to see we can navigate through our partials,
clicking links updates the document's location anchor, and refreshing or visiting
the URL directly shows the correct content.... Except that we notice a slight
flicker of the content area.   Since we have to wait for the page to be loaded in
order to bind our Parjax plugin there is a variable amount of time the initial
content is show before our AJAX request completes.

### Fixing that menacing flicker
Hmmm... Perhaps the user won't notice?    I mean it is almost instantaneous on my localhost tests.
No, no, no, we can't have this on our conscious.   The thing is it will *never* be
faster than our localhost tests.  Every other user accessing our page will occur network
latency and they will know we let it slide.   Not on our watch, we need to fix it.

The correct solution here is to add a css rule that hides the content until the page is loaded.
With `display: none;` added to our CSS the Parjax element is now hidden when the page loads.

Now all we need to do is show the element when it has the correct content.  Sounds like we need a callback on our `$container.load()` method; this should do the trick:

```

    // detect a deep link
    if (document.location.hash.length > 0) {

      // remove the # from the fragment identifier and make an ajax request to load the content
      $container.load(document.location.hash.substr(1) + ' [data-parjax-container] *', function() {

        // we have the correct content loaded, show it.
        $container.show();

      });
    }


```

The last thing to do is to show the element when there isn't a document identifier present since the plugin will only invoke the callback when it requests the initial partial.
Easy enough, we already have the conditional, so let's add our else statement.


```

    // detect a deep link
    if (document.location.hash.length > 0) {

      // remove the # from the fragment identifier and make an ajax request to load the content
      $container.load(document.location.hash.substr(1) + ' [data-parjax-container] *', function() {

        // we have the correct content loaded, show it.
        $container.show();

      });

    } else {

      // we did not have to load any content, show immediately
      $container.show();

    }


```

That's it!   We're good.  Now we have a Parjax plugin we can use.   You might be familiar enough with jQuery to
know it includes a few predefined animations.  Since have the hooks in place,
let's swap a `fadeIn()` call for our `show()` methods, but only the one showing after a user clicks.   **Fancy.**


```
// jquery.parjax.js
(function($, document) {

  $.fn.parjax = function() {
    return this.each(function() {
      $container = $(this);
      var baseUrl = document.location.protocol + '//' + document.location.host;

      $container.on('click', 'a', function(event) {

        // only use Parjax when we are requesting our own pages
        if (this.host === document.location.host) {

          // hiding will prevent DoS from the impatient.
          $container.hide();

          // load the link via ajax and set the response into our Parjax element
          $container.load($(this).prop('href') + ' [data-parjax-container] *', function() {

            // we have the content loaded, show it.
            $container.fadeIn();

          });

          // persist the link that was clicked
          document.location.hash = this.href.replace(baseUrl, '');

          // don't let the browser do its thing
          event.preventDefault(true);
        }
      });

      // detect a deep link
      if (document.location.hash.length > 0) {

        // remove the # from the fragment identifier and make an ajax request to load the content
        $container.load(document.location.hash.substr(1) + ' [data-parjax-container] *', function() {

          // we have the correct content loaded, show it.
          $container.show();

        });

      } else {

        // we did not have to load any content, show immediately
        $container.show();

      }

    });
  }

})(jQuery, document);
```

### What's up next
In my next post, we'll step it up a notch and support multiple Parjax containers with persistence.   I bet you can't wait.







