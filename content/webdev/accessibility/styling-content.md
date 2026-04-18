---
title: "Styling content"
linkTitle: "Styles"
weight: 60
# description:
---

Many of these styles focus on *progressive enhancement*, which is a strategy of building layers of styles that activate based on the browser's capabilities. A browser that supports a newer feature gets the enhanced experience. A browser that does not falls back to a functional baseline. Here are some links about progressive enhancement:

- [It's about time I tried to explain what progressive enhancement actually is](https://piccalil.li/blog/its-about-time-i-tried-to-explain-what-progressive-enhancement-actually-is/)
- [Understanding Progressive Enhancement](https://alistapart.com/article/understandingprogressiveenhancement/)
- [Quick Tips for High Contrast](https://sarahmhigley.com/writing/whcm-quick-tips/)
- [WHCM and System Colors](https://adrianroselli.com/2021/02/whcm-and-system-colors.html)

## Color

Do not rely on color alone to convey information. Approximately 8% of men and 0.5% of women have some form of color vision deficiency. A red/green error indicator, for example, is completely invisible to users with red-green color blindness. Always pair color with a second cue such as an icon, a text label, a pattern, or a shape.

Do not use color alone to do the following:

- Indicate an action
- Prompt a response
- Visualize a state change
- Distinguish a visual element

### Color contrast

Test your colors with a contrast checker: [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/).

Color contrast is measured in ratios developed by the W3C. They range from 1:1 (same color) to 21:1 (black on white). Here are W3C's **minimum** requirements:

- Regular text: `4.5:1`
- Regular text at 24px+: `3:1`
- Bold text at 19px+: `3:1`
- No requirements for purely decorative elements like logos, brand names, and similar items

## Respecting user preferences

Modern operating systems expose user preferences like dark mode, reduced motion, and forced colors. You can read these preferences in CSS with `@media` query at-rules and adapt your styles accordingly. This lets users configure their experience at the system level once and have it respected everywhere.

The following at-rules cover the most common preferences:

```css
@media(prefers-color-scheme: dark) { ... }      /* detect system dark mode */
@media(prefers-contrast: more) { ... }          /* increased contrast */
@media(forced-colors: active) { ... }           /* detect forced-color mode */
@media(inverted-colors: inverted) { ... }       /* detect whether colors are inverted */
@media(scripting: enabled) { ... }              /* detect whether JS is enabled */
@media(prefers-reduced-transparency) { ... }    /* remove transparency */
```

A key use case is `prefers-reduced-motion`. Users with vestibular disorders, epilepsy, or motion sensitivity can be harmed by large-scale animations. Wrap any animation that moves significant amounts of the screen in this query:

```css
@media (prefers-reduced-motion: no-preference) {
  .hero-banner {
    animation: slide-in 0.5s ease-out;
  }
}
```

This applies the animation only when the user has not requested reduced motion. The banner loads without animation for users who need that accommodation.

## Preserve semantic information

Some CSS properties affect the semantic meaning of an element, which can change how a screen reader identifies it.

### Buttons and links

- Be careful with `display: contents;`. Buttons or links with `display: contents;` cannot accept keyboard focus.
- Links with zero dimensions (no height, width, or padding) do not accept keyboard focus.

### Tables

The following CSS patterns strip semantic information from tables, which breaks screen reader announcements of column and row counts:

- `display: contents;` removes all semantic information from tables, table rows, table headers, and table cells in Safari.
- Flexbox or grid removes all semantic information from tables in Safari.
- `display: none;` on the `<caption>` element may remove the accessible name from the table.

### Forms

It is common to hide native form elements or labels and build custom replacements. Do not apply the following properties to hide them, or they will not appear in the accessibility tree:

- `display: none;`
- `visibility: hidden;`

`appearance: none;` removes semantic meaning in some screen readers and Firefox.

### Lists

Setting `list-style: none;` removes the semantic `list` role in Safari with VoiceOver. This is a known browser bug. If the visual result still looks like a list and you want to preserve the semantics, set `list-style-type: '';` instead. An empty string removes the bullet marker without signaling to Safari that the list is no longer a list.

### Interactive elements

`pointer-events: none;` prevents mouse activation but does not remove the element from the keyboard tab order. Users can still focus the element with the keyboard but cannot activate it with `Enter`, which creates a confusing and broken experience.
