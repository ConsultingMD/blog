---
layout: post
title: "Keeping jQuery Datepicker in Place on Mobile"
date: 2014-07-29 10:44:18 -0700
comments: true
categories:
---

While scrolling on a page with jQuery datepicker open, the datepicker normally behaves fixed at a certain position on the page. However, when using it in a mobile app, this is not what we see. The datepicker begins scrolling with the page and appears to be fixed at a certain position on the screen as opposed to the page. It becomes out of sync with the input box and floats on the screen.

After inspecting the datepicker, we discover `ui-datepicker-div` is appended to the end of the body. The datepicker thinks its position is relative to the screen and not the page which is what we want. We try two solutions to fix this problem. One, move the datepicker into a div relative to the panel of the page scroll and two, hide the datepicker when the page is scrolled.

In the first solution, with some experimentation, if the datepicker, `ui-datepicker-div`, is moved elsewhere, let’s say after the input field, the datepicker becomes fixed on the page and no longer scrolls with the page.

This can be done by using `appendTo` after the datepicker is binded to the input field.

``` javascript
@$('#input_field_name').datepicker({dateFormat: $.datepicker.RFC_2822, maxDate: 0})
@$('#ui-datepicker-div').appendTo('#input_field_name')
```

A new problem surfaces, the datepicker is now in the wrong position on the page. Depending on how far you have scrolled down and where your input field is when you selected it, the calendar’s position is fixed to the page but at a position relative to the screen which is almost like the opposite of our original problem.

Our first solution does not solve our problem, thus, we try our second solution where the datepicker disappears upon scrolling. With this solution, we no longer have to move the datepicker around.

By using javascript, we can hide the datepicker on scroll. However, the scroll event is not called when we scroll on mobile. If we were to use touchmove, when you click on the input field, the datepicker would appear then disappear immediately. Hence, the easiest way would be to use `onBlur`. When you begin to scroll on a mobile app, the input field blurs automatically. Using this to our advantage, we specifically call a function when the class `.hasDatepicker` blurs.

``` javascript
$(window).scroll(), ->
```

``` javascript
$(window).bind 'touchmove', ->
```

``` javascript
$('.hasDatepicker').on 'blur', ->
  $('.hasDatepicker').datepicker( "hide" )
```

You'll notice another problem appears! If you try to click on the prev or next button to get to a different month, the datepicker hides and will not allow you to change months. When the prev or next button is clicked, the input field is blurred for a short moment before it is focused again. Therefore, the datepicker hides as soon as it detects the input field has been blurred. Additionally, if you click anywhere on the calendar, the datepicker will disappear as well.

By determining if the calendar was clicked, we can hide the datepicker when the input field blurs if the calendar was not clicked.

Putting all of this in a function and calling it, we have two functions inside that function. In the first function, we can determine if the calendar was clicked, and in the second function, we hide the datepicker. We begin by initializing the var `calendarActive` outside of the two functions so it can be accessed in the second function. On `mousedown` (this works for mobile as well), we determine if the calendar was clicked. If you try

``` javascript
$(event.target).hasClass('ui-datepicker')
```

you always receive false no matter where you click on the calendar. `$(event.target)` is more specific than we expect it to be and the unselectable parts and prev/next buttons do not share any classes in common when you look at `event.target.className`. But they share the same parent class so we we check if the parent has the class ui-datepicker.

``` javascript
$(event.target).parents().hasClass('ui-datepicker')
```

Lastly, as we did before, we want to hide the datepicker on blur if `calendarActive` we just set is false.

Putting it all together:

``` javascript
hideDatePicker = ->
  calendarActive = false
  $(window).on 'mousedown', (event) ->
    calendarActive = $(event.target).parents().hasClass('ui-datepicker')
    true

  $('.hasDatepicker').on 'blur', ->
    $(@).datepicker('hide') if calendarActive == false

hideDatePicker()
```

Upon blurring the input field, when scrolling or clicking outside of the calendar, the datepicker hides.
