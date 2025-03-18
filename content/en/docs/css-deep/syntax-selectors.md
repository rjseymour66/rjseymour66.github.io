---
title: "Syntax and selectors"
# linkTitle: "CSS in Depth"
weight: 20
# description:
---


CSS is a domain-specific, declarative programming language. _Domain-specific_ means that it is a specialized language that solves a specific problem. It is a _declarative language_ because it tells the browser what to do, rather than how to do it.

## Syntax

Much of how to accomplish stylings with CSS depends on the context, so it is best to understand the priniciples of the language:

declaration
: A line of CSS that includes a property and a value. Invalid declarations are ignored.

declaration block
: A group of declarations inside curly braces that is preceded by a selector

ruleset | rule
: Selector plus declaration block is a ruleset or rule. Rule is less common.

at-rule
: Rules that begin with the `@` symbol, like `@import` or `@media`.


```css
/* ruleset */
selector {
  /* declaration block */
  property: value; /* declaration */
  property: value;
}

/* Example */
body {
  color: black;
  font-family: Helvetica;
}
```

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

_External CSS_ groups styles together in an external `.css` stylesheet. Link the stylesheet in the `<head>` tag of the website. Name the stylesheet `styles.css`:

```css
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```
The `<link>` element uses the following attributes:
- `rel`: Describes the relationship between the HTML document and what you are linking to it.
- `href`: Stands for hypertext reference and indicates where to find the document that we want to include.

## Devtools 

For each element, devtools lists styles by specificity. Styles at the top override those below them, and they are crossed out. The ruleset location in the stylesheet is to the right of the style.

## Stylesheet origins

There are 3 differnt stylesheets that _cascade_---allow stylesheets to overwrite or inherit from one another---when we apply styles to HTML:
- User-agent stylesheet: Stylesheet internal to your web browser (i.e. Google Chrome). These include:
    - blue, underlined links 
    - discs for lis 
    - margins
    - font size
- Author stylesheet: Stylesheet developed and applied by a developer to HTML with a link tag.
- User stylesheet: Custom user-provided stylesheets, which are usually created and applied to overcome accessibility issues.

### Cascade

The following list describes the order that styles cascade, in order of precedence:

1. `!important` user-agent
2. `!important` user
3. `!important` author
4. Normal author
5. Normal user
6. Normal user-agent

### Resolving rule conflicts

Cascade determines how rule conflicts are resolved. When there is a conflict, the cascade uses the following hierarchy to resolve it:
1. Stylesheet origin: Where the stylesheet comes from
   - Browser applies user-agent styles, then User styles to override
   1. Author stylesheet - what the CSS dev adds to the page. Also called "User styles"
   2. User stylesheet - added via browser extension, usually added to overcome a11y issues
   3. User-agent - Browser's default styles
      1. headings h1 - h6
      2. p tag top and bottom margins
      3. ol and ul styles
      4. link colors
      5. default font sizes
2. Inline styles: Whether the style is applied to an HTML element directly.
   <a href="/" style="color: white;"></a>
   - Can override inline styles with author `!important` rule
3. Layer: Which layer is the style defined in? Uses the last layer.
4. Selector specificity: Which selector has highest specificity.
5. Scope: Use declaration with innermost scope.
6. Source order: Declaration that comes latest in source order.

## CSS reset

[meyerweb CSS reset](https://meyerweb.com/eric/tools/css/reset/)

When you start a project, you want to make sure that you have a consistent baseline across browsers---you do not want user agent stylesheets to impact your CSS. 

To create this baseline, you apply a CSS reset that undoes the browser's default styles. To apply a CSS reset, you can create a `reset.css` stylesheet to add to the project, and link it before your author stylesheet to ensure it is applied before your custom styles:

```css
<head>
  <link rel="stylesheet" href="reset.css">
  <link rel="stylesheet" href="styles.css">
</head>
```

In production, you want to add CSS resets with one of the following methods:
- Copy the reset styles to the beginning of your stylesheet.
- Use a reset CDN. After the user loads your page (or another page that uses the same CDN) once, they will cache the styles.

## Specificity

Always keep specificity as low as you can so there are no issues in the future.

When multiple styles are applied to one element, the browser must determine which style to apply. When a declaration "wins" the cascade, it is called the _cascaded value_.

1. If a selector has more IDs, it wins 
2. If 1 is a tie, the selector with the most classes wins 
3. If 2 is a tie, the selector with the most tag names wins 

Pseudo-class selectors and attribute selectors have the same specificity as a class selector:

```css
/* pseudo-class */
link:hover {...}

/* attribute */
[type=input] {...}
```

Universal selector and combinators do not impact specificity scores.


### Specficity notation 

Count the number of ids, classes, tags, and notation and write the total in `<id>, <class>, <tag>` notation:

| Selectors                        | IDs  | Classes | Tags | Notation |
| :------------------------------- | :--- | :------ | :--- | :------- |
| `html` `body` `header` `h1`      | 0    | 0       | 4    | 0,0,4    |
| `body` `header.page-header` `h1` | 0    | 1       | 3    | 0,1,3    |
| `.page-header` `.title`          | 0    | 2       | 0    | 0,2,0    |
| `#page-title`                    | 1    | 0       | 0    | 1,0,0    |


### Reducing specificity
You can use the `:where()` pseudo-class to reduce specificity (it has 0 specificity). `:where(.nav)` is equivalent to `.nav`, but has a specificity of 0.

## Special values

These values manipulate the cascade and allow you to undo styles that you set elsewhere in the stylesheet. This is helpful for something like flexbox, whose properties have multiple keyword values, and remembering the original values is difficult:

- `inherit`: Causes the element to inherit the value from its parent. When you want to use inheritance when a cascading value is preventing it. This is helpful when you need to control a value in one place. For example, the background color is controlled only in the parent element.
- `initial`: Resets to the default value for the property (not the element) when there are applied styles that you want to undo. This overrides all styles from both author and user-agent stylesheets. For example, you can set `background-color` to `initial` instead of `transparent`, bc that is its default value. Other common uses:
  - `border: initial;` (default is `none`)
  - `width: initial;` (default is `auto`)
- `unset`: When applied to an inherited property, it sets the value to `inherit` , when applied to non-inherited property, it sets it to `initial`. This overrides all styles from both author and user-agent stylesheets.
- `revert`: Overrides your author-styles but preserves the user-agent styles.



### Common rules 

1. Don't use IDs in selectors 
2. Don't use `!important`

### Inherited styles

CSS applies the following general rules to determine inherited styles:
- **Theme-related styles**, such as color and font size, are generally inherited
- **Layout considerations** are generally not inherited.

Anything defined in a rule overrides an inherited value.

If there is no cascaded value for an element, it might inherit a value. A common inherited value is applying `font-family` to the `body` element so you do not have to apply it to each element in the `body`. Here is a non-comprehensive list:

- color
- font
- font-family
- font-size
- font-weight
- font-variant
- font-style
- line-height
- letter-spacing
- text-align
- text-indent
- text-transform
- white-space
- word-spacing.
- list-style
- list-style-type
- list-style-position
- list-style-image


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

Some common examples:

```css
/* try not to use font shorthand bc it is easy to omit values and use initial values */
font: font-style font-weight font-size line-height font-family;

background: background-color background-image background-size background-repeat background-position background-origin background-chip background-attachment;

border: border-width border-style border-color;

border-width: top right bottom left;
```

> Be aware of shorthands that silently override other styles. When you omit values for shorthand values, it sets the omitted values to their `initial` value.

When you add only 3 values, it applies to the top, right, then bottom, and the left value takes.

> Generally better to have more padding on sides of buttons, less on top.

The following properties specify horizontal left/right values, then top/bottom. This is because they represent a Cartesian grid, which uses x, y ordering:

- background-position
- box-shadow
- text-shadow

> Tip: Property that specifies two measurements from a corner, think “Cartesian grid.”
> Property that specifies measurements for each side all the way around an element, think “clock.”

## Vendor prefixes

Vendor prefixes aren't as common as they used to be, but some browsers still require them for a few properties:
- `-webkit`: Safari or Chrome
- `-moz`: Mozilla Firefox
- `-o`: Opera
- `-ms`: IE and older versions of Edge

Automate this with [autoprefixer](https://github.com/postcss/autoprefixer), and [Lightning CSS](https://lightningcss.dev/).


## Progressive enhancement

This means that your stylesheet can apply styles that are not supported by some browsers. As user browsers upgrade, the features become available (future-compatible).

Use a new feature with the `@supports` feature query. This allows you to use a feature that is possibly not supported, but you must provide a supported fallback style, too.

```css
.coffees {
  margin: 20px 0;
}
 
/* fallback styles */
.coffees > a {
  display: inline-block;
  min-width: 300px;
  padding: 10px 15px;
  margin-right: 10px;
  margin-bottom: 10px;
  color: black;
  background-color: transparent;
  border: 1px solid gray;
  border-radius: 5px;
}

/* feature query */
@supports (display: grid) {
  .coffees {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 10px;
  }
 
  .coffees > a {
    margin: unset;
    min-width: unset;
  }
}
```

Example usage:

```css
/* Only apply rules in the feature query block if the queried declaration isn’t supported */
@supports not(<declaration>)

/* Apply rules if either queried declaration is supported */
@supports (<declaration>) or (<declaration>)

/* Apply rules only if both queried declarations are supported */
@supports (<declaration>) and (<declaration>)

/* Apply rules only if the given selector is understood by the browser (for example, @supports selector(:user-invalid)) */
@supports selector(<selector>)
```

## Input elements

The following selects all `input` elements that are not checkboxes and not radio buttons:

```css
input:not([type:checkbox]):not([type:radio]) {
    display: block;
    width: 100%;
    margin-top: 0;
}
```

For input elements, the width is determined by the size attribute, which is the number of characters it should contain without scrolling. Use the `width` attribute to force a specific width value.