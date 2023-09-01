---
title: "Sass"
linkTitle: "Sass"
weight: 3
description: >
  Getting started with Sass.
---

Sass is a preprocessor that is compiled into CSS. Preprocessors add functionality to CSS and require a build step.

## Installing and starting the processor

You can install and manage Sass with `npm`:

```bash
$ npm install -g sass
$ npm start
```

After you start npm, it recognizes any Sass files in the project directory and watches for changes.

When you write Sass code, there is a `filename.css.map` file that serves as a source map for the browser's developer tools. This file tells the browser where the CSS code originated in the preprocessed file.

## Syntax

Use SCSS, which is a superset of CSS that allows any valid CSS in addition to Sass features.

## Getting started

The browser uses the compiled CSS, so in the `<head>` tag, link the processed `.css` file:

```html
<head>
    <link rel="stylesheet" href="styles.css">
</head>
```

## Features

### Variables

Variables are one of the reasons that CSS preprocessors became popular. They use the following syntax:

```css
$primary: value;
```

Sass variables do not understand the DOM, and they do not use cascade or inheritance. They are block-scoped, so be sure to declare them globally unless you only want to use them within a single ruleset.

You can redefine the same variable within the same `.scss` file. If you change a variable value, rules defined before the change use the initial value, while rules defined after the change use the newer value.

### @extend

`@extend` allows you to create a base rule that we can point to from different rules. You can create defaults and then apply them to other selectors without duplicating your code:

```css
.image-base {
  width: 300px;
  height: 300px;
  object-fit: cover;
  margin: 0 2rem;
}

img:first-of-type { @extend .image-base }
img:nth-last-of-type(2) { @extend .image-base }
img:last-of-type { @extend .image-base }
```

When you use the `@extend` keyword, Sass doesn't copy or generate code, it adds the selector to the base.

### @mixin

Mixins allow you to generate declarations and rules. They can accept parameters---like functions---and return styles. Parameters begin with a `$`, just like variables:

```css
@mixin handle-img($border-radius, $position, $side) {
  border-radius: $border-radius;
  object-position: $position;
  float: $side;
  margin-#{$side}: 0;
}
```

To use the mixin, use the `@include` keyword, followed by the mixin name and arguments:

```css

img:first-of-type { 
  @extend .image-base; 
  @include handle-img(20px 100px 10px 20px, center, left);
}
```

The `@include` keyword is different that `@extends` because it generates code, which is helpful for setting dynamic properties.

### Interpolation

Interpolation lets you parameterize a parameter. For example:

```css
padding-#{$direction}: 10rem;
```
In the previous example, you can define a `$direction` variable that takes the place of the `#` character. This is similar to template strings in Javascript or formatted strings in Python.

### Nesting

