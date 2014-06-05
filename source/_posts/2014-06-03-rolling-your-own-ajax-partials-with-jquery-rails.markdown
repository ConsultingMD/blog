---
layout: post
title: "Rolling your own Ajax Partials with jQuery and Rails"
date: 2014-06-03 17:15:06 -0700
comments: true
categories: frontend parjax jquery
---

With the advent of Rails 4 we were introduced to a new featured called [Turbolinks](https://github.com/rails/turbolinks).  Turbolinks is a gem that optimizes browser rendering by bypassing the need for the browser to recompile styles and scripts for every page.  This is done by using Ajax to make requests to the server and only updating the body and title of the page to bypass rendering of any assets included in the HTML head.

[Pjax](https://github.com/defunkt/jquery-pjax) is a similar gem to Turbolinks but instead of rendering the entire page pjax renders partials and replaces only designated elements within the page with the response from the server.  Pjax has an advantage over Turbolinks in that it does not require the server to render or deliver the entire page to the browser.  This can be a very benifitial enhancement for clients that constrained by bandwidth or processing power limitations.

But what we wanted to employ some of these optimizations but didn't want to leverage a gem.  Often times it is beneficial to build a tailored solution and in this case we'll do it in about 20 lines of code.

### Getting started

First to get off to a good start let's create a new [jQuery plugin](http://learn.jquery.com/plugins/) which will encapsulate our functionality.  Here we will start with a simple jQuery plugin template that defines as `parjax` within the jQuery function namespace.

``` javascript
// jquery.parjax.js
(function($) {

  $.fn.parjax = function() {
    return this.each();
  }

})(jQuery);
```

We now have the ability to invoke the plugin on a set of jquery elements, using the familiar jQuery selector based chaining syntax.

Let's do this by adding the following to our controller's client code.  In our example, it's `part1.js.coffee` and we will enhance all anchor elements to our new plugin.

``` javascript
# part1.js.coffee
$ ->
  $('a').parjax()
```

Now to the fun part, implementing the functionality.  Our initial requirement is is to **intercept and override a user clicking a link** and **set the server response into a specific document element** without re-rendering the entire web page.

### Overriding the default Anchor behavior

Intercepting a user click is jQuery 101 and very straightforward to add to our plugin.  Since we progressively enhancing our web page when we intercept the click, we want to cancel this browser's default behavior.  jQuery gives us a simple method of doing this with the `event.preventDefault()` method.

``` javascript
// jquery.parjax.js
(function($) {

  $.fn.parjax = function() {
    return this.each(function() {

      $(this).on('click', function(event) {
        event.preventDefault(true);
      });

    });
  }

})(jQuery);
```

Now if we reload our page, what we have essentially done is remove the default browser behavior for clicking links.  Nothing will happen when a user clicks on a link and the browser supports JavaScript.

### Fetching the server response and setting it into the document.

Again jQuery gives us a method to do what what we want.  We'll use jQuery's [.load()](http://api.jquery.com/load/) method to get the server response and set it into the DOM.   As part of our progressive enhancement we will reuse the anchor tag's existing href value.

The only unknown at this point is *which* element we are going to replace with the server's response.  Turbolinks operates by replacing the entire `BODY` element with the server response, while pjax deals strictly with replacing the contents of a specific element.   We are going to go the pjax route as it will provide us with more flexibility in the long run (and it is a much easier implementation since we won't have to parse the body content out of the server response, perhaps we will come back to this later).

What we need here is an element designation which defines it as a receiver or container for the server response.  We'll use a [data attribute](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes) for our container element.   Let's not go willy-nilly here though, we want do some future proofing, so let's define a data attribute namespace for our Parjax decorations. `data-parjax-container` sounds right.  Since this attribute defines a truthy value we'll keep our markup succinct and follow the HTML practice of the presence of the attribute to imply a value of true.

Let's designate the element in our layout by adding the `data-parjax-container` attribute to the containing element around our `yield` in `application.html.erb`.

``` html
<!DOCTYPE html>
<html>
<head>
  <title>Parjax!</title>
  <%= stylesheet_link_tag "application", :media => "all" %>
  <%= javascript_include_tag "application" %>
  <%= csrf_meta_tags %>
</head>
<body>
  <h1>Parjax Example</h1>
  <div data-parjax-container>
    <%= yield %>
  </div>
</body>
</html>

```

Now that we know which element we are going to replace the contents of we can go ahead and finish the code to implement our requirements.

The additional magic here is we'll also specify the optional selector for the load method to parse the server response and only select the appropriate HTML from the response to insert into the document.

Since the server is returning the entire HTML page, we would like to discard anything outside of the element context (our `data-parjax-container` element defined in the layout).  This prevents the browser from having to re-render any scripts and styles that are defined by the layout providing us with the optimization we are going after by implementing this functionality.

``` javascript
// jquery.parjax.js
(function($) {

  $.fn.parjax = function() {
    return this.each(function() {

      $(this).on('click', function(event) {

        $(this).closest('[data-parjax-container]')
          .load($(this).prop('href') + ' [data-parjax-container] *');

        event.preventDefault(true);
      });

    });
  }

})(jQuery);
```

A refresh and a click on a link now provides us with our intended result, an Ajax request which replaces a container with the a partial from the response from the server.

### A Bug Emerges
Further testing though shows a flaw in our plugin in that any link returned in a partial does not exhibit our Parjax behavior.

This originates from how we were binding our plugin.  We incorrectly bound Parjax to anchor tags and since these can be removed and added from the DOM by Parjax they lose the added functionality.

We are either forced to rebind the plugin every time an Ajax request is completed or change our plugin binding.  I vote for changing the plugin binding.

The following modifications change our context from the link to the container.

``` javascript
// part1.js.coffee
$ ->
  $('[data-parjax-container]').parjax()
```

``` javascript
// jquery.parjax.js
(function($) {

  $.fn.parjax = function() {
    return this.each(function() {
      $container = $(this);

      $container.on('click', 'a', function(event) {
        $container.load($(this).prop('href') + ' [data-parjax-container] *');
        event.preventDefault(true);
      });

    });
  }

})(jQuery);
```

### Current Limitations

We have implemented Ajax partials in Rails in under 20 lines of code.   Although very lean, it is not very robust and should only be used as is in certain situations.

The main limitation is the state of the application not being preserved by the browser.  If the user refreshes the page they will essentially "start over".  We will address this in a future blog post by implementing [pushState()](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history) support and exploring fallbacks for browsers that do not support this feature.

We will also touch on implementing plugin options, transition animations, support for request failures, form submissions, redirects, and nested parjax views.  We will also look into further optimization including how to respond with only the rendered partial instead of the entire web page.

With the current limitations Parjax should only be used in its current state for auxiliary application content like a lazy loading carousel or an inline help dialog, etc.

**But stay tuned, we are just getting started.**
