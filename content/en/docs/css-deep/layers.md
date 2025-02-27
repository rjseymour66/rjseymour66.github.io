---
title: "Cascade Layers"
linkTitle: "Layers"
weight: 110
# description:
---

> If the browser does not support layers, then all rules in that layer are supported. Check at https://caniuse.com. There is also a polyfill for layers if there is no support: https://www.oddbird.net/2022/06/21/cascade-layers-polyfill/

Layers let you group and partition your styles and assign a priority order to them so that that you can control which styles take precedence. **Styles that are not in a layer take precedence over styles in a layer**.

This solves a problem where you might want a ruleset with lower specificity to take precedence over others. For example, if you have a style that selects all links, but you also have a button link that you want to style differently without increasing the specificity:

```css
/* 0, 1, 1 specificity */
a:any-link {
    color: red;
}

/* 0, 1, 0 specificity */
.button-link {
    color: blue;
}
```

## Syntax

Use the `@layer` at-rule to define a layer. The layer that occurs later in the stylesheet takes precedence over layers that appear before them. To solve the problem described above:

```css
@layer global {
    a:any-link {
        color: blue;
        font-weight: bold;
    }
}

@layer theme {
    .button {
        display: inline-block;
        padding: 0.5rem;
        color: white;
        background-color: blue;
        font-weight: normal;
        text-decoration: none;
    }
}
```

### Order and priority

**Styles that are not in a layer take precedence over styles in a layer. If you use layers, place all styles inside a layer. Unlayered styles are good for debugging.**

You should decide the priority order of layers and declare them upfront. Here, layers are declared in a single line at the top of the stylesheet in increasing order of precedence:

```css
@layer reset, lowest, higher, highest;

@layer reset {...}
@layer lowest {...}
@layer higher {...}
@layer highest {...}
```

Otherwise, layers are prioritized by where they occur in the stylesheet. You can define a layer, and then add styles to that layer later in the stylesheet without changing its priority--its priority is already established by the first `@layer` clause.

In this example, the `lowest` layer still has the lowest priority even though it appears twice, after another stylesheet:

```css
@layer lowest {
    /* styles */
}

@layer higher-than-lowest {
    /* styles */
}

@layer lowest {
    /* additional lowest styles */
}
```

### !important and priority

Layer priority is reversed when you add `!important` to a layer. For example, `!important` styles in the `reset` layer take precedence over `!important` styles in the `highest` layer. This works because you usually apply the most general styles to lower-priority layers, so you want these `!important` styles to apply across layers:

```css
@layer reset, lowest, higher, highest;
```

### Nesting layers

Nesting one or more layers within a layer provides more granular control. For example, you can import a stylesheet and the importing stylesheet respects any layers defined in the imported stylesheet.

You can nest layers with indentation, or you can use dot notation. Dot notation is easier to reference at a later time:

```css
/* identation notation */
@layer components {
    @layer first {...}
    @layer second {...}
}

/* dot notation */
@layer components.first {...}
@layer components.second {...}
```

### revert-layer and all

The `revert-layer` keyword removes any styles applied by the author styles and restores the user-agent styles. Use this if you have a global styles layer and want to override these styles on a higher-priority layer:

```css
@layer global, theme;

@layer global {
    a:any-link {
        font-weight: bold;
    }
}

@layer theme {
    .main-content a:any-link {
        font-weight: revert-layer;
    }
}
```

If you don't want to revert each property individually, you can use the `all` property. `all` accepts these values:
- `initial`
- `inherit`
- `unset`
- `revert`
- `revert-layer`

These styles revert all layer properties on an element:

```css
@layer global, theme;

@layer global {
    a:any-link {
        font-weight: bold;
    }
}

@layer theme {
    .main-content a:any-link {
        all: revert-layer;
    }
}
```

### Anonymous layers

You don't have to name layers--they are still applied using stylesheet order--but you should to help reference them later and for general flexibility:

```css
@layer {
    /* styles */
}

@layer {    /* these styles are applied */
    /* styles */
}
```

### Assign stylesheet to layer

You can import and assign a stylesheet to a layer:

```css
@import url("styles.css") layer(base);

@layer base {
    /* additional styles */
}
```

## Organization

Layers are new, but there are organizational patterns you can use. Here are recommended naming conventions and structure:

```css
@layer reset, theme, global, layout, modules, utilities;
```
They have the following order of precedence:
1. `utilities`: short reusable styles
2. `modules`: reusable units
3. `layout`: primary page structure
4. `global`: universal fonts, colors, etc. Sometimes called the `base` layer.
5. `theme`: custom property colors
6. `reset`: adjust user-agent styles
7. (Optional) `states`: Dynamically-added styles depending on the state or status of a module. You can also add them in the `modules` layer.
   Ex: states of a dropdown menu

The subsequent sections provide brief examples of the types of styles you can include in each layer.

### reset

Developers include a reset stylesheet to fix inconsistencies between default user-agent styles. This is not much of an issue anymore, but it is helpful to start at a baseline and apply styles from there.

Add these styles to all projects as a reset:

```css
@layer reset {
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  body {
    margin: unset;
  }

  button,
  input,
  textarea,
  select {
    font: inherit;
  }

  img,
  picture,
  svg,
  canvas {
    display: block;
    max-inline-size: 100%;
    height: auto;
  }

  @media (prefers-reduced-motion) {
    *,
    *::before,
    *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
}
```

### theme

This layer should set these styles:
- custom properties: Set site-wide custom properties here. The priority is lower so you can easily override them at a higher-priority layer.
- `accent-color`: Sets the accent color for elements like checkboxes, radio buttons, range inputs, progress bars
- `color-scheme`: Set to `light` or `dark` so the browser (user-agent) can determine good default styles for form inputs and scrollbars.

Here is a sample of how to set this up. A real site would use many more custom properties:

```css
@layer theme {
  :root {
    --brand-color: #0063cc;
    --background-color-1: #edf3fa;
    --background-color-2: #c6cdd5;
    --foreground-color-1: #edf3fa;
    --foreground-color-2: #c6cdd5;
    --font-main: "Helvetica Neue", Arial, sans-serif;
    --font-heading: Georgia, sans-serif

    accent-color: var(var(--brand-color))
    color-scheme: light;
  }
}
```

### global

Set all the default values that you want applied through the site:
- font styles
- background colors
- font colors
- form inputs and labels
- code blocks
- blockquotes
- link states like `:hover`

```css
@layer global {
  :root {
    font-size: clamp(1rem, 0.4rem + 0.8svw, 1.2rem);
  }

  body {
    font-family: var(--font-main);
    background-color: var(--background-color-1);
    color: var(--foreground-color-1)
  }

  a:any-link {
    color: var(--brand-color)
  }

  h1 {
    font-family: var(--font-heading);
    font-size: 2.2rem;
  }

  h2 {
    font-size: 1.125rem;
  }

  @media (min-width: 768px) {
    h1 {
      font-size: 3rem;
    }
    h2 {
      font-size: 2rem;
    }
  }
}
```

### layout

Defines styles that structure a page from the outside in--the high-level page layout:
- header
- footer
- sidebars
- home page, article layouts

Layer priority means that you do not have to worry about specificity with layers, so you can use IDs in layers because of the override behavior. **Only use IDs if you plan to use them once on the stylesheet. If you need to reference them elsewhere, use classes.**

```css
@layer layout {
  #homepage {
    display: grid;
    grid-template-areas: 
      "header header"
      "main sidebar"
      "footer footer";
    grid-template-columns: 1fr 300px;
    gap: 1rem;
  }

  #homepage > header,
  #homepage > footer {
    grid-column: span 2;
  }

  #article > main {
    max-inline-size: 1000px;
    margin-inline: auto;
  }
}
```

### modules

Modules are reusable units, also called components, blocks, or objects. Examples include:
- dropdown menus
- modals
- banner images
- nav bars
- information cards

```css
@layer modules {
  .nav-menu {                   /* Nav menu module */
    margin-block: unset;
    padding-inline: unset;
    border: 1px solid #ccc;
    list-style: none;
  }

  .nav-menu > li + li {
    border-top: 1px solid #ccc;
  }

  .nav-menu > li > a {
    display: block;
    padding: 0.8em 1em;
    color: inherit;
    font-weight: normal;
  }

  .nav-menu > li > a:hover {
    color: var(--brand-color);
    background-color: white;
  }

  .card {                       /* Card module */
    padding: 1rem;
    border-radius: 0.5rem;
    background-color: #fff;
  }

  .card > h3 {
    align-self: end;
    margin-block: 0;
    padding-block-end: 0.5rem;
    border-block-end: 1px solid #eee;
  }
}
```

### utilities

Utility classes do a single, specific thing like center text or hide an element. These rulesets generally contain one declaration. They are quick helpers that you don't want to be overridden. You probably won't need more than a dozen of these:

```css
@layer utilities {
  .text-center {
    text-align: center;
  }

  .hidden {
    display: none;
  }

  .border-radius {
    border-radius: 1rem;
  }
}
```

## :is() and :where()

### :is()

`:is()` is a pseudo-class that takes one or more arguments as a selector, then applies rules to all selectors that match:
- Specificity is derived from whichever argument has the highest specificity
- Does not work with pseudo-element selectors, like `::before`.
- If you include a selector that is not supported in a browser, then `is:()` still works on selectors that are supported.

```css
.contact-form input,
.contact-form textarea,
.contact-form select {
    padding: 5px 10px;
}

/* is equivalent to */
.contact-form :is(input, textarea, select) {
    padding: 5px 10px;
}
```

### :where()

The same as `:is()`, but always reduces the specificity to zero. Useful if you want to select an element based on ID but don't want to override other styles on the same layer:

```css
/* 0,0,1 specificity */
:where(#login-form) input

/* 0,0,1 */
a:where(:any-link) {
    color: blue;
}

/* 0,1,0 */
.button {
    color: red;
}
```

## Nesting

You can nest styles in CSS like you can SASS. The child rule is an additional selector on the parent rule:
- Specificity is determined by the final, joined selector
- Avoid nesting too many levels because it creates too high a specificity

```css
.card {
    padding: 1rem;
    background-color: #fff

    > h3 {
        margin: 0 aut;
    }

    .card-body {
        padding: 0 1em;
    }
}
```

### Nesting selector (&)

Lets you create a compound selector with nesting.

When you nest a selector, it creates a descendant selector by default (`selector1 selector2`) with a space in between. You might want to create a compound selector (`selector1.selector2`). Use a `&` in the selector:
- can target psuedo classes and psuedo elements
- derives its specificity from the highest specificity of the selectors

```css
/* equivalent to .modal.is-open */
.modal {
    display: none;

    &.is-open {
        display: block;
    }
}

.button {
    background-color: blue;

    &:hover {
        background-color: red;
    }

    &::after {
        content: 'test';
    }
}

/* 1,1,1 specificity, even if input:focus is matched */
input,
#login-form button {
    &:focus {
        border-color: #ccc;
    }
}
```

Behind the scenes, the browser warps the parent selectors in `:is()` and appends the child selector. The following two rulesets are equivalent:

```css
button,
input,
textarea,
select {
  &.invalid {
    border: 1px solid red;
  }
}

:is(button, input, textarea, select).invalid {
  border: 1px solid red;
}
```

### Media queries

Nest media queries to keep related code together:

```css
h1 {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 2.2rem;

  @media (min-width: 800px) {
    font-size: 2.6rem;
  }

  @media (min-width: 1200px) {
    font-size: 3rem;
  }
}
```

You can also nest `@layers` and `@supports` rules.