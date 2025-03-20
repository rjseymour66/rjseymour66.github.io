---
title: "HTML"
# linkTitle: "CSS in Depth"
# weight: 20
# description:
---

There are non-replaced and replaced HTML elements:

- non-replaced elements are standard elements with opening or closing tags that usually display text.
- replaced elements are replaced by objects, such as a raster image or UI widget. The browser gives them a default appearance, so it is difficult to style some of them effectively.

There are also void elements, which is a subset of replaced elements. These have only one tag and cannot contain text or nested elements. Placing a slash before the closing bracket is outdated and unnecessary. Examples include:

- `<br>`
- `<col>`
- `<embed>`
- `<hr>`
- `<img>`
- `<input>`
- `<link>`
- `<meta>`
- `<source>`
- `<track>`
- `<wbr>`

Other replaced elements, like `<iframe>` are not void because they can contain other elements or text.

## Attributes

Attributes provide information about the element and only appear in the opening tag.

- most are name/value pairs
- boolean attributes sometimes don't even require true or false
- always quote the attribute value
- attribute values are case sensitive, especially when used with CSS or JS

## Appearance

All browsers have agent stylesheets that style the elements. Because many of them might vary, you need to select the correct element for what you are trying to accomplish. This is why we have "semantic" elements, which are helpful for assistive technologies. Each tag name has a special meaning and is directly related to its role in the page.

## DOM and JS

The DOM is the data representation of the page structure. When the browser parses the DOM, it creates a JS object for every element and section of text, which are called element and text nodes.

The [HTML DOM API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API) lets you access and control every HTML element through the DOM. This is possible through the [HTMLElement interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement), which gives you access to attributes like `id` and `class`, and lets you handle events like input, pointer interactions, transitions, and animations. Each HTML element inherits the HTMLElement interface, and some elements have specialized interfaces for their unique interactions.
