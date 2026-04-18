---
title: "Lists"
# linkTitle: ""
weight: 80
# description:
---

These are the main kinds of lists:

- unordered lists: `<ul>`
- ordered lists: `<ol>`
- description lists: `<dl>`

Unordered and Ordered lists can only have `<li>` elements as a child.

## Ordered lists

By default, ols use numbers for their marker. You can change the marker with either the `list-style-type` CSS property or the `type` attribute:

- If you remove the type, consider adding `role="list"` to the list if it is important to know that this acts like a list

```html
<ol type="A">
  <li>Blender</li>
  <li>Toaster</li>
  <li>Vacuum</li>
</ol>
```

OLs have three element-specific attributes:

- `type`: sets the numbering type. This accepts these values:
  - `1`: default
  - `A`: letters
  - `i`: lowercase roman nums
  - `I`: uppercase roman nums
- `reversed`: reverses the order of the numbers
- `start`: sets the starting value, default is 1

## List items

Can be a child of:

- `<ol>`
- `<li>`
- `<menu>`

Accepts the `value` integer--it overrides the value of the `<ol>` `start` attribute, if there is one

- use this if the number is important semantically. Otherwise, use [CSS counters](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_counter_styles/Using_CSS_counters) with the `::marker` pseudo-element

If you don't include the closing tag, the browser will add one when it encounters the next `<li>` or list-type element (e.g. `</ol>`).

## Description lists

`<dl>` used to be called "definition lists". Includes these elements:

- `<dt>`: description terms
- `<dd>`: description details

```html
<dl>
  <dt>Blendan Smooth</dt>
  <dd>Originally a margarita maker, they are now an aspiring load balancer.</dd>
  <dt>Toasty McToastface</dt>
  <dd>
    Formerly partially to fully baked, they are now an aspiring nuclear codes
    handler.
  </dd>
</dl>
```

You can also nest these elemetns in a `<dl>`:

- `<div>`
- `<template>`
- `<script>`
