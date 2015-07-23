---
layout: post
title: "Javascript Computed Height Nuances"
date: 2015-07-20 19:27:09 -0700
comments: true
categories: 
author: Timothy Wang
---

A few weeks ago, I discovered an interesting bug that highlighted subtleties in the way Javascript calculates height for tags.

On our dashboard for doctors, we have a piece of javascript called 'fittable' that fires on each browser resize. This function is responsible for maintaining the same amount of spacing between our content and the bottom of the browser.

```
$(window).resize ->
    $('.panel .tab-content').fit()
  .resize()
```

```
jQuery.fn.fit = ->
  @.each ->
    $el = $(@)
    $current = $el
    height = $(window).height()
    while $current[0].tagName != 'BODY'
      $current.siblings().each ->
        height -= $(@).outerHeight(true) if $(@).css('float') == 'none'
      $current = $current.parent()
      height -= $current.outerHeight(true) - $current.height()
    $el.outerHeight(height)
```

The 'tab-content' div is enclosed within a parent container that's visible to the user due to border styling.  The intent here was that 'tab-content' should always be set to a height such that its parent has a consistent amount of spacing between its bottom border and the browser window. Fittable attaches itself to 'tab-content', and on every browser resize, it calculates a new height for 'tab-content'.  At first, the height of 'tab-content' is set to the total height of the browser.  Fittable then recursively iterates over tab-content's parent, and its parent, until it reaches the body tag. On each iteration it subtracts the outerHeight of the element from its height, removing all margin and padding from tab-content's final height. It also removes the height of any siblings that are nonfloat elements.  At the end of the loop, it sets the new height of the tab-content div.

What was strange was that the browser would consistently set a lower-height for 'tab-content' than expected.  Somewhere in this loop, extra pixels were being subtracted from 'tab-content's calculated height.  The first time the page loaded, elements would be sized correctly, but upon resize, the elements would be misaligned.  Even when I manually fired fit() through the console, 'tab-content' would still be missing height.

To debug, I manually stepped through the Javascript and inspected the height value on each loop. Finally, I discovered the source of the missing height. I had several Chrome plugins that had added script tags inside the body. These tags would surprisingly evaluate to a positive height value. Therefore, fittable would subtract this height from tab-content’s calculated height, even though these tags were invisible! Therefore, tab-content's height would be consistently short.

To get around this, I modified fittable so it only subtracted the sibling tag's height if the element was a non-float, non-script, and visible element to the user.

```
height -= $(@).outerHeight(true) if $(@).css('float') == 'none' and $(@)[0].tagName != 'SCRIPT' and $(@).is(":visible")
```

Moral of the story, be careful about script tags. It's recommended to put script tags in the head for a reason - I learned the hard way that script tags can break DOM layouts.  What's worse is that I had no control over these script tags being inserted into the DOM - my plugins were responsible for this.  When you work with Javascript, you have to be prepared for factors out of your control.
