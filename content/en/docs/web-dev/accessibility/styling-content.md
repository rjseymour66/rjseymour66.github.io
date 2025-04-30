---
title: "Styling content"
linkTitle: "Styles"
weight: 60
# description:
---

Many of these styles focus on progressive enhancement, which is when you build layers of styles that turn themselves on based on the browser's capabilities. Here are some links about PE and some others:
- [It's about time I tried to explain what progressive enhancement actually is](https://piccalil.li/blog/its-about-time-i-tried-to-explain-what-progressive-enhancement-actually-is/)
- [Understanding Progressive Enhancement](https://alistapart.com/article/understandingprogressiveenhancement/)
- [Quick Tips for High Contrast](https://sarahmhigley.com/writing/whcm-quick-tips/)
- [WHCM and System Colors](https://adrianroselli.com/2021/02/whcm-and-system-colors.html)

## Color

Do not use color alone to do the following:
- indicate an action
- prompt a response
- visualize a state change
- distinguish a visual element

### Color contrast

Test your colors with a contrast checker: [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/).

Color contrast is measured in ratios developed by the W3C. They range from 1:1 (same color) to 1:21 (black on white). Here are W3C's **minimum** requirements:

- Regular text: `4.5:1`
- Regular text at 24px+: `3:1`
- Bold text at 19px+: `3:1`
- No requirements for purely decorative elements like logos, brand names, etc.

## Respecting user preferences

Use CSS `@media` query at-rules to address user system and browser preferences:

```css

@media(prefers-color-scheme: dark) { ... }      /* detect system dark mode */
@media(prefers-contrast: more) { ... }          /* increased contrast */
@media(forced-colors: active) { ... }           /* detect forced-color mode */
@media(inverted-colors: inverted) { ... }       /* detect whether colors are inverted */
@media(scripting: enabled) { ... }              /* detect whether JS is enabled */
@media(prefers-reduced-transparency) { ... }    /* remove transparency */
```

## Preserve semantic information

Some CSS properties affect the semantic meaning of an element, which can impact how a screen reader identifies the element.

### Buttons and links

- Be careful with `display: contents;`. Buttons or links with `display: contents;` cannot accept keyboard focus.
- Links with 0 dimensions (no height/width/padding) do not accept keyboard focus.

### Tables

- `display: contents;` removes all semantic information from tables, table rows, table heads, and table cells on Safari
- Flexbox or grid removes all semantic information from tables on Safari
- `display: none;` on the `<caption>` element might remove accessible names

### Forms

It is common to hide native form elements or labels and build your own. Don't use these properties to hide them, or they will not display in the accessibility tree:
- `display: none;`
- `visibility: hidden;`

`appearance: none;` removes the semantic meaning in some SRs and Firefox.

### Lists

Setting `list-style:none;` removes the semantic meaning for a list. This should be fine if the final product does not resemble a list. If you want to maintain the semantic meaning, you can set `list-style-type: '';`, which is equal to an empty string.

### Interactive elements

`pointer-events: none;` means that the user cannot activate it with a mouse click or by pressing `Enter`. They can still focus with the keyboard.