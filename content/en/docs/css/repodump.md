---
title: "Repo dump"
linkTitle: "Repo dump"
weight: 1
description: >
  Notes about basic Repo dump.
---


# Notes

CSS uses _late binding_: the content and its styles are not pulled together until after the authoring of both is complete.

The _root node_ is the ancestor of all elements in the document. It is `<html>`. `:root` is its special pseudo-class selector.

Define `line-height` on the body element so that it is inherited by rest of the document.

The original intention of vendor prefixes was to allow developers to experiment with technology before they used it in production. But developers started using it in production immediately. 

Use [Autoprefixer CSS online](https://autoprefixer.github.io/).

Shorthand properties:
```css
.item {
    /* Apply to all four sides */
    margin: 1em;
    margin: -3px;

    /* vertical | horizontal */
    margin: 5% auto;

    /* top | horizontal | bottom */
    margin: 1em auto 2em;

    /* top | right | bottom | left */
    margin: 2px 1em 0 auto;
}
```

### Feature queries

Set fallbacks for CSS features that might not yet be supported by all browsers. Use the `@supports` rule:

```css
@supports (display: grid) {
    ...
}

@supports (declaration) or (declaration) {
    ...
}

@supports (declaration) and (declaration) {
    ...
}
```
In the previous example, if the browser supports the grid layout, it applies the styles in the parentheses.

### Document flow
Don't set height on a container unless in special circumstances. Normal document flow is designed to work with a constrained width and unlimited height. The height of a container is determined by its contents, not the container.

When you set an element's height, you are in danger of _overflowing_ the container.

If you expect overflow, set it to `auto` so that the browser manages when to add scrollbars automatically.

Adjacent top or bottom margins combine (_collapse_) to form a single margin. Flex items do not collapse.

Always set up the a `flow` class using the lobotomized owl to target any element that immediately follows any other element:

```css
.flow * + * {
    margin-top: 1.5rem;
}
```



## Nav styling

When creating menu items:
- Apply padding to the internal `<a>` tags to provide more clickable surface area.
- Use `display: block;`. This allows its parent to derive the height from their padding, not line height.

Horizontal nav styling:

```css
.ul-class {
    display: flex;
    margin: 0;
    padding: .5em;
    background-color: #5f4b44;
    list-style-type: none;
    border-radius: .2em;
}

.ul-class > li {
    margin-top: 0;    
}

.ul-class > li > a {
    display: block;
    padding: .5em 1em;
    background-color: #cc6b5a;
    color: #fff;
    text-decoration: none;
}

.ul-class > li + li {
    margin-left: 1.5em;
}
```








