---
layout: post
title: "CSS Best Practices"
date: 2015-12-21 18:14:36 -0800
comments: true
categories: css, css3
author: Bashir Eghbali
---
examples in slim.

## Use a grid system

** DO ** consistently use a grid layout where contents on the page are layed out in increments of the smallest
block in the grid. try [bootstrap](http://getbootstrap.com/css/#grid). example below uses bootstrap 3's 12 grid layout.

```
  .row
    .col-md-4
      Name
    .col-md-8
      input type=text
```

** DO NOT ** modify position (top, left, right) and margins to layout your content

## Use relative sizes

** DO ** use ems to specify size for fonts, margins and paddings. You can set the par values at the document level and everything else will just scale relatively
```
.restaurant {
  font-size: 1.5em;
}
```
** DO NOT ** use px to specify size. it breaks responsiveness

## Be Specific

** DO ** specify accurate/unambiguous selectors. The order of specificity is inline styles, ids, attributes/classes and elements.
```
.restaurant > div:first-child {
  font-weight: bold;
}
```
** DO NOT ** use important!. It is almost never needed and is often used because people fail to be specific in their styles so their intended rules don't apply to the elements they want.
** DO NOT ** use ids for styling. ids are specific to the contents of the dom and less its layout. Styles should be general and reusable and not tied to ids.

## Use classes sparsely
** DO ** use existing dom elements for styling instead of adding a class to every dom element that needs styles
```
.restaurant > div:first-child {
  font-weight: bold;
}
```
instead of
```
.restaurant .name {
  font-weight: bold;
}
```
** DO NOT ** use class for every element you want to style. There is probably a natural order of elements to reference it
** DO NOT ** use very long class names. class names add up and significantly increase the asset size that needs to be loaded by a page, degredating performance

## Modularity
** DO ** use mixins to build modular visual components that are reusable
```
@mixin circle($width) {
  width: $width;
  height: auto;
  @include border-radius($width);
  border: 0.2em white solid;
}
```
and you can reference them in other mixins as well
```
@mixin avatar($width) {
  @include circle($width);
  height: $width;
  border: none;
  background-position: center top;
}
```

** DO NOT ** copy and paste styles. Whenever you see yourself do that, create a more general class or a mixin.

## Even precision
** DO ** use precisions that are even splits of 100. This will help with responsiveness and an even layout
```
margin: 1.25em;
padding: 1.125em;
```
** DO NOT ** use random precisions and attempt to match a mock in pixel perfect form, this will throw your entire layout in disarray

Shameless footer: We would love smart engineers to join us to change the world of healthcare. inquire [here](https://www.grandrounds.com/careers/)
