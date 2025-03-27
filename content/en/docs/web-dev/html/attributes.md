---
title: "Attributes"
# linkTitle: ""
weight: 60
# description:
---


Attributes provide details about the functionality of the element. They are always in the opening tag:

- global attributes can be used in any element
- some attributes apply to some elements but not all
- element-specific attrs apply to a single element
- Boolean attrs do not require a value

## Case-sensitive

You should always quote attribute values:

- Attrs that are part of the HTML spec are case-insensitive
- Values that you assign to the attrs are case-sensitive

```html
<!-- the type attribute is case insensitive: these are equivalent -->
<input type="text" />
<input type="TeXt" />

<!-- the id attribute is case sensitive: they are not equivalent -->
<div id="myId">
  <div id="MyID"></div>
</div>
```

## Booleans

If a Boolean attribute is present, it is considered true. Examples:

- `autofocus`
- `inert`
- `checked`
- `disabled`
- `required`
- `reversed`
- `allowfullscreen`
- `default`

If you add a string value to a Boolean attr, it resolves to `true`, no matter what the value. These are all equivalent:

```html
<input required />
<input required="" />
<input required="yay" />
```

### JS to toggle

Rather than change the markup when a value changes from true to false, use JS to toggle the value:

```js
const myElement = document.getElementById("myElement");
myElement.removeAttribute("muted");
myElement.setAttribute("muted");
```

## Enumerated attributes

If you are unsure if an attr is Boolean or enumerated, check. Enumerated attrs might have different default values if not defined properly.

Enumerated attributes have a limited set of predefined valid values. They have default values if the attribute is present but the value is missing (`<style contenteditable>` = `<style contenteditable="true">`):
- omitting the attr doesn't mean that its false
- a present attr with a missing value isn't always true
- default value for invalid values isn't a null string
  - default values are attr-specific
  - if you add an invalid value, then the default is applied
- in most cases, missing and invalid values are the same

## Global attributes

Set [global attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes#list_of_global_attributes) on any HTML element, but they may not have an effect on some elements. For example, you can't apply `hidden` to the `<meta>` tag.

### id

Defines a unique identifier for an element. Has several uses cases:
- target of a link fragment
- handle for JS scripting
- associate form element with a label
- label or description for assistive technologies
- targeting styles with high CSS specificity

Some general guidelines:
- `id`s are case-sensitive
- should be unique within the document
- `id`s should have no spaces, or you have to escape the spaces in your HTML, CSS, and JS.
- only use ASCII letters, digits, `-`, and `_`
- first char should be a letter
- come up with your own naming convention

#### Link fragments

If the `<a>` href attr starts with a `#`, then it links to an element with an `id` that matches the string that follows the `#` character.

```html
<nav>
  <a href="#reg">Register</a>
  ...
</nav>

...

<section id="reg">
  <h2>Machine Learning Workshop Tickets</h2>
</section>
```

To scroll to the top of the page, use the built-in `href=#top` or `href=#`.

#### CSS selectors

Target elements with one of these syntaxes:
- `#id-name`
- `[id="id-name"] for less specificity

#### Scripting

Either grab by ID or with querySelector methods and the `#`:

```js
const switchViaID = document.getElementById("switch");
const switchViaSelector = document.querySelector("#switch");
```

#### `<label>`

The `for` value for the `<label>` element should be the same as the `id` value of the form control that it is associated with:
- a label can be associated with one form control, but a form control can have multiple associated labels

```html
<label for="minutes">Send me a reminder</label>
<input type="number" name="min" id="minutes">
<label for="minutes">before the workshop resumes</label>.
```
You can also nest an input within a label. This means that you don't need to use the `for` or `id` attributes because they are inherently linked:

```html
<label>
  Send me a reminder <input type="number" name="min"> before the workshop resumes
</label>.
```

You should use the `for` and `id` with a separate label. It helps with screen readers, and any label associated with a form control is clickable or tappable, which helps usability.

Turn an element into a region landmark by referencing another element's `id` with the `aria-labelledby` attribute:

```html
<section id="about" aria-labelledby="about_heading">
<h2 id="about_heading">What you'll learn</h2>
```
### class

Lets you target an HTML element with CSS or JS. Takes a space-separated list of case-sensitive classes.

Properly structuring your document helps you correctly select elements with CSS without unneeded nesting.

### style

Not recommended. Use a stylesheet.

Lets you apply inline CSS styles. Descendant elements inherit the property unless overridden by other settings. It only accepts a single ruleset, so you can't apply keyframes or at-rules.

### tabindex

Enables an element to receive focus--it adds the element to the tab order. Takes an integer as a value, with the following behavior:
- negative value (`-1`) makes the element receive focus, but does not add it to the tabbing sequence
- `0` makes it focusable and adds to the tabbing sequence in its source code order
- (not recommended) `1` (or any number > `1`) puts the element in a prioritized focus sequence. This can create a bad user experience, because the focus jumps around the document in a non-default order

By default, these elements receive focus and are in the tabbing sequence, as if they have `tabindex=0` set:
- Form controls
- links
- buttons
- content editable elements

### role

Part of the [ARIA specification](https://w3c.github.io/aria/#introroles).

Gives semantic meaning to content so screen readers can explain an elements expected user interaction. It doesn't change how the browser presents the element, or how the keyboard or pointer devices interact with it--it tells a screen reader that a non-semantic element is being used as a semantic element. It is taking on a semantic element's role.

### contenteditable

When set to true, the element is editable, focusable, and added to the tab order as if `tabindex=0` is set.
- only supports `true` or `false` (is enumerated)
- invalid values default to `inherit`

These are all equal:

```html
<style contenteditable>
<style contenteditable="">
<style contenteditable="true">
```

Each element has a `contentEditable` property that you can control with JS:

```js
const editor = document.getElementById("myElement");
if(editor.contentEditable) {
  editor.setAttribute("contenteditable", "false");
} else {
  editor.setAttribute("contenteditable", "");
}
```

## Custom attributes

Create a custom attribute with the `data-` prefix. These are a great way to communicate application-specific information with JS.
- custom name cannot begin with `xml` or include a colon (`:`)



```html
<blockquote data-machine-learning="workshop"
  data-first-name="Blendan" data-last-name="Smooth"
  data-formerly="Margarita Maker" data-aspiring="Load Balancer"
  data-year-graduated="2022">
  HAL and EVE could teach a fan to blow hot air.
</blockquote>
```

Access custom attributes with the `dataset[<name>]` syntax:

```js
el.dataset["machineLearning"]; // workshop
e.dataset.machineLearning; // workshop
```

The `dataset` property returns a `DOMStringMap` obj of each element's `data-` attribute:

```js
for (let key in el.dataset) {
    // do something with dataset
}
```