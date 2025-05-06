---
title: "Secrets"
# linkTitle: ""
weight: 5
# description:
---

## Vendor prefixes

Here are some helpful links to determin if you need an autoprefixer:
- [autoprefixer](https://github.com/postcss/autoprefixer)
- [-prefix-free](https://projects.verou.me/prefixfree/)

## Tips

### currentColor

Considered the first ever variable in CSS, this value is equal to the `color` property. The `color` property is inherited. This means that it applies either the `color` value for the current element or--if `color` is not defined--the initial color for that property.

### (avoid) media queries

> Now, you can use _container queries_.

Try to avoid media queries, when possible. You want to exhaust every possible way to make the website flexible without resorting to media queries. If you do use them, let the content dictate the breakpoints, not an arbitrary device.

Here are some tips to avoid them:
- Use percentages instead of fixed widths
- Use `max-width` instead of `width` so you can adapt to smaller viewports
- Always use `max-width: 100%;` for replaced content such as `img`, `object`, `video`, and `iframe`. Replaced elements are elements that are replaced by resources that the browser fetches.
- Use `column-width` instead of `column-count` so you can get one column on smaller resolutions.

### Use shorthands

Longhand properties only apply to that specific property--you don't reset all the other properties that might affect your styles. For example:

```css
body {
    background: red;            /* background will always be red */
    background-color: red;      /* other background-* props might affect the background */
}
```

You can also combine shorthand and longhand declarations to reduce the amount of edits. Here, we use the shorthand for the `url()` fallbacks, and the specific properties for specific styles:

```css
body {
    background: url(some.png) no-repeat top right,
                url(some.png) no-repeat bottom right,
                url(some.png) no-repeat bottom left;

    background-size: 2em 2em;
    background-repeat: no-repeat;
}
```

In the preceding example, you can add all the fallbacks to the `background` property, and add the individual styles to the other props. If you need to make an edit to `*-size` or `*-repeat`, then you edit only those properties.

## Backgrounds and borders

### Translucent borders

By default, borders extend underneath the border area, so it is hard to see transparent borders on elements with lighter backgrounds. To fix this, use the `background-clip` property to clip the background at the `padding-box`, rather than the `border-box`:

```css
.translucent-border {
  border: 20px solid hsla(0, 0%, 100%, 0.5);
  background: white;
  background-clip: padding-box;
}
```

### Multiple borders

The original fix was to use multiple `box-shadow` properties, but that is bad for multiple reasons:
- Shadows don't take up layout space, so you have to add additional margin to make the doc flow correctly
- Shadows can't capture mouse events

The fix is to use `outline`:

```css
.multiple-border {
  background: yellowgreen;
  border: 10px solid #655;
  outline: 5px solid deeppink;
}
```

You can also add `outline-offset` to add space between the border and the outline. Here, we use a negative value to make the outline look like stitching:

```css
.inner {
  background: yellowgreen;
  border: 10px solid #655;
  outline: 1px dashed deeppink;
  outline-offset: -24px;
}
```

### Flexible background positioning