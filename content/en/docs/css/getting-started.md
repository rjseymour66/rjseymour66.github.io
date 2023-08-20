---
title: "Getting started"
linkTitle: "Getting started"
weight: 1
description: >
  Getting started with CSS.
---

CSS is a domain-specific, declarative programming language. _Domain-specific_ means that it is a specialized language that solves a specific problem. It is a _declarative language_ because it tells the browser what to do, rather than how to do it.

## Syntax

Each CSS style is called a _rule_, and each rule uses the following format:

```css
selector {
  property: property-value;
}
```

The `property: property-value;` is called a _declaration_, and the declarations with the curly braces are called a _ruleset_. You can omit the semicolon (`;`) from the last declaration in the ruleset. Invalid declarations are ignored.

## Adding CSS to HTML

You can add CSS to a file with the following methods:

Inline
: Add the CSS to the HTML tag.

Embed
: Add the CSS to the page within the `<style>` tag, which is nested in the `<head>` tag.

External
: Link a stylesheet to the HTML with the `<link>` tag.

In nearly all cases, you add CSS to a project with an external stylesheet.

### External CSS

_External CSS_ groups styles together in an external `.css` stylesheet. Link the stylesheet in the `<head>` tag of the website:

```css
<head>
  <link rel="stylesheet" href"styles.css">
</head>
```
The `<link>` element uses the following attributes:
- `rel`: Describes the relationship between the HTML document and what you are linking to it.
- `href`: Stands for hypertext reference and indicates where to find the document that we want to include.

## Cascading stylesheets

There are 3 differnt stylesheets that _cascade_---allow stylesheets to overwrite or inherit from one another---when we apply styles to HTML:
- User-agent stylesheet: Stylesheet internal to your web browser (i.e. Google Chrome)
- Author stylesheet: Stylesheet developed and applied by a developer to HTML with a link tag.
- User stylesheet: Custom user-provided stylesheets, which are usually created and applied to overcome accessibility issues.

The following list describes the order that styles cascade, in order of precedence:

1. User stylesheet with `!important` declaration.
2. Author stylesheet with `!important` declaration.
3. User stylesheet.
4. Author stylesheet.
5. User agent stylesheet.

## CSS reset

[meyerweb CSS reset](https://meyerweb.com/eric/tools/css/reset/)

When you start a project, you want to make sure that you have a consistent baseline across browsers---you do not want user agent stylesheets to impact your CSS. 

To create this baseline, you apply a CSS reset that undoes the browser's default styles. To apply a CSS reset, you can create a `reset.css` stylesheet to add to the project, and link it before your author stylesheet to ensure it is applied before your custom styles:

```css
<head>
  <link rel="stylesheet" href"reset.css">
  <link rel="stylesheet" href"styles.css">
</head>
```

In production, you want to add CSS resets with one of the following methods:
- Copy the reset styles to the beginning of your stylesheet.
- Use a reset CDN. After the user loads your page (or another page that uses the same CDN) once, they will cache the styles.

## Specificity

When multiple styles are applied to one element, the browser must determine which style to apply.

### Inherited styles

CSS applies the following general rules to determine inherited styles:
- **Theme-related styles**, such as color and font size, are generally inherited
- **Layout considerations** are generally not inherited.

Anything defined in a rule overrides an inherited value.

### Specificity values

When browsers evaluate competing styles, they assign the following values and apply the style with the highest score:

- 100: ID selectors (`#id-name`)
- 10: Class selectors (`.class-name`), attribute selectors (`[type="submit"]`), and psuedo-classes (`::`)
- 1: Type selectors (`button`)
- 0: Universal selector (`*`)

## CSS selectors

### Type
Targets the HTML element by name.
`h1`, `p`, `li`

### Class
Targets the HTML element by associated `class` attributes:
```
// index.html
<h1 class="class1 class2 class3">Heading</h1>

// stylesheet
.class1 {...}
```

### ID
Unique attribute that is generally used for Javascript. Each ID is unique to the web page.

```
// index.html
<div id="id-name"></div>

// stylesheet
#id-name {...}
```

> You should avoid applying styles with an ID attribute.


### Combinators

Combinators combine multiple selectors, which allows more complex CSS without overusing class or ID names. You apply styles using relationships between elements.

#### Descendant (space)

Targets all HTML elements within a parent. The target descendant element does not have to be an immediate child:

```css
<parent> <descendant> { <declaration> }
article p { font-size: 2rem; }
```

#### Child (`>`)

Targets the immediate child elements of a particular selector:

```html
<ul class="list">   // parent
  <li></li>         // child
    <ul>            // grandchild
      <li></li>     // great-grandchild
      <li></li>
      <li></li>
    </ul>
  <li></li>         // child
  <li></li>
</ul>
```

To apply styles, use the `>` operator to select the child elements. Note that color is an inherited property, so you have to use the descendant selector to revert the styles on the `grandchild` elements:

```css
.list > li { color: red }
.list > li ul { color: initial }
```

#### Adjacent (`+`)

Style sibling elements at the same level, directly after another:

```css
header + p { font-size: 2rem }
```

The adjacent combinator is useful when displaying errors in HTML forms.

#### General sibling (`~`)

Targets all elements that are siblings after the element targeted by the selector. The second element does not need to immediately follow the first, like the adjacent sibling combinator.

The following example targets all sibling `<img>` tags that is preceded by a `<header>` element. In other words, the browser locates the `<header>` element, then applies the styles to all `<img>` elements after that element:

```css
header ~ img { ... }
```

### Pseudo elements

These selectors target a state or parts of an element that might not exist yet.

#### Pseudo-class

[Pseudo-classes](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes)

A _pseudo-class_ is added to a selector to target a specific state of the element. This is common for elements that the user interacts with, such as links, buttons, and forms.

To define styles for a pseudo-class, use a colon (`:`). The following example styles link elements according to state:

```css
/* href is not in browser history */
a:link {
  color: #000
}

/* href is in browser histor */
a:visited {
  color: #111
}

/* cusor is over element */
a:hover {
  color: #222
}

/* element that receives keyboard events by default */
a:focus {
  color: #333
}
```

#### Pseudo-elements

[List of pseudo-elements](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-elements#list_of_pseudo-elements)

Allow you to style a specific part of an element. To define styles for a pseudo-element, use a double-colon (`::`). You can use a single colon, but that is for backward compatibility only.

For example, the following rule styles only the first letter of a paragraph that is adjacent to the header element:

```css
header + p::first-letter {
  color: red;
}
```

### Attribute selectors

This selector targets HTML elements that have a specific attribute with the same value. This is commonly used to style links and forms.

The following example places the word "pre" before the tag in square braces:

```css
[lang="en"]::before {
  content: "pre"
}
```

### Universal selector

The universal selector (`*`) applies to all HTML elements. It has a specificity of 0, so you can override it easily.

For example, the following rule changes the font-family to sans-serif for all `h1` child elements:

```css
h1 > * {
  font-family: sans-serif;
}
```

## Shorthand 

Shorthand lets you replace multiple CSS properties with one property. A common example is `padding`, which merges the following into a single property:
- `padding-top`
- `padding-right`
- `padding-bottom`
- `padding-left`

You can combine all of the above properties with the `padding` property, listing the values clockwise from the top or combining top and bottom:

```css
h1 {
  padding: top right bottom left
}

h1 {
  padding: top-bottom left-right
}
```
