---
title: "Preprocessors"
# linkTitle: ""
weight: 210
# description:
---

A preprocessor takes the source file that you write and translates it into an output file, which is a standard CSS stylesheet. A preprocessor doesn't add features to CSS, but it makes it easier to write.
- SCSS uses bracket syntax and the `.scss` extension


## Installation and setup

```bash
npm init -y                         # create new npm project
npm install --save-dev sass         # install sass and add to package.json
mkdir sass build                    # create two dirs
touch sass/index.scss               # create main scss file
vim package.json                    # create build scrip
  ...
  "scripts": {
    "sass": "sass sass/index.scss build/styles.css"
  },
  ...
npm run sass                        # run build script, [over]write build/styles.css 
```
When you run `npm run sass`, it creates a `styles.css.map` source map file that maps code in the `.css` output file to the `.scss` source file.

## Features

### Variables

Variables are one of the reasons that CSS preprocessors became popular. They use the following syntax:

```css
$primary: value;
```

Sass variables do not understand the DOM, and they do not use cascade or inheritance. They are block-scoped, so be sure to declare them globally unless you only want to use them within a single ruleset.

You can redefine the same variable within the same `.scss` file. If you change a variable value, rules defined before the change use the initial value, while rules defined after the change use the newer value.

### Inline arithmetic

You can use `+`, `-`, `*`, `/`, and `%` in a declaration to compute values. This feature is what inspired the `calc()` function in CSS:

```css
$padding-left: 3em;

.note-body {
  padding-left: $padding-left * 2;
}

/* output.css */
.note-body {
  padding-left: 6em;
}
```
### Nested selectors

You can nest selectors inside declaration blocks to group related code. Nesting increases specificity, so be avoid extensive nesting.

The preprocessor prepends the top-level selector to the nested selector with a space between (`.parent .nested`) to create a descendant selector. To omit the space and create a compound selector (`.parent.nested`), use an `&`:

```css
.site-nav {
  display: flex;

  > li {
    margin-top: 0;

    &.is-active {
      display: block;
    }
  }
}

/* output.css */
.site-nav {
  display: flex;
}
.site-nav > li {
  margin-top: 0;
}
.site-nav > li.is-active {
  display: block;
}
```

You can also nest `@media` queries. This is helpful when changing selectors--you don't have to change the selector in the media query, too:

```css
html {
  font-size: 1rem;
  @media (min-width: 45em) {
    font-size: 1.25rem;
  }
}
/* output.css */
html {
  font-size: 1rem;
}
@media (min-width: 45em) {
  html {
    font-size: 1.25rem;
  }
}
```

### Partials (@use)

Partials let you split your code into separate files for better code organization. The preprocessor will concatenate partial files together so the browser only needs to request one file.

To create a partial, start the file name with an underscore. For example, `_file.scss`. Then, import the partial at the beginning of your main `.scss` file with the `@use` at-rule. For more information about how you can use the `@use` at-rule, see the [Sass docs](https://sass-lang.com/guide/#modules).

Here is a simple example:

```css
/* _button.scss */
.button {
  padding: 1em 1.25em;
  background-color: #265559;
  color: #333;
}

/* index.scss */
@use "button";

html {
  font-size: 1rem;
  @media (min-width: 45em) {
    font-size: 1.25rem;
  }
}

/* output.css */
.button {
  padding: 1em 1.25em;
  background-color: #265559;
  color: #333;
}

html {
  font-size: 1rem;
}
@media (min-width: 45em) {
  html {
    font-size: 1.25rem;
  }
}
```
### Mixins

A mixin is a small, reusable chunk of CSS. Use this if you have some commonly repeated rules, or a font style that you need to repeat in multiple places throughout the stylesheet.

Because mixins duplicate code, they compress well with `gzip`, which is how you should compress all network traffic.

Define a mixin with the `@mixin` at-rule, and use it with an `@include` at-rule. The `@include` at-rule tells the preprocessor to generate code:

```css
@mixin box {                                  /* mixin defition */
  border-radius: 5px;
  box-shadow: 5px 5px 10px rgb(0 0 0 /0.1);
}

.main-tile {
  @include box;                               /* apply mixin */
  color: #333;
}

/* output.css */
.main-tile {
  border-radius: 5px;
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.1);
  color: #333;
}
```

Mixins can also accept parameters---like functions---and return styles. Parameters begin with a `$`, just like variables:

```css
@mixin alert-variant($color, $bg-color) {
  padding: 0.3em 0.5em;
  border: 1px solid $color;
  color: $color;
  background-color: $bg-color;
}

.alert-info {
  @include alert-variant(blue, lightblue);
}
.alert-danger {
  @include alert-variant(red, pink);
}

/* output.css */
.alert-info {
  padding: 0.3em 0.5em;
  border: 1px solid blue;
  color: blue;
  background-color: lightblue;
}

.alert-danger {
  padding: 0.3em 0.5em;
  border: 1px solid red;
  color: red;
  background-color: pink;
}
```

To use the mixin, use the `@include` keyword, followed by the mixin name and arguments:

```css

img:first-of-type { 
  @extend .image-base; 
  @include handle-img(20px 100px 10px 20px, center, left);
}
```

### Extend

The `@extend` at-rule is similar to a mixin, but it doesn't copy declarations--it groups selectors and places them at an earlier location in the stylesheet. Because `@extend` moves the selector to an earlier location in the stylesheet, it might affect the cascade:

```css
%message {
  padding: 0.3em 0.5em;
  border-radius: 0.5em;
}

.message-info {
  @extend %message;
  color: blue;
  background-color: lightblue;
}

.message-danger {
  @extend %message;
  color: red;
  background-color: pink;
}

/* output.css */
.message-danger, .message-info {
  padding: 0.3em 0.5em;
  border-radius: 0.5em;
}

.message-info {
  color: blue;
  background-color: lightblue;
}

.message-danger {
  color: red;
  background-color: pink;
}
```


---

<!-- ******************************************** -->


## Getting started

The browser uses the compiled CSS, so in the `<head>` tag, link the processed `.css` file:

```html
<head>
    <link rel="stylesheet" href="styles.css">
</head>
```

## Features



### @extend





### Interpolation

Interpolation lets you parameterize a parameter. For example:

```css
padding-#{$direction}: 10rem;
```
In the previous example, you can define a `$direction` variable that takes the place of the `#` character. This is similar to template strings in Javascript or formatted strings in Python.

### Nesting

