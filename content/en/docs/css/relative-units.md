---
title: "Relative Units"
linkTitle: "Relative Units"
weight: 7
description: >
  Notes about basic Relative Units.
---

Once upon a time, developers knew the exact screen size that their application used. Now, users can set their screen to any size, and the browser has to calculate units and apply styles.

This means that you have to create style rules that work in any context. This is called responsive design.

_Responsive design_ is when styles respond differently based on the size of the browser window.

## Relative units

Ems and rems are the most common (only?) relative units in CSS. Values declared using relative units are evaluated by the browser to an absolute value.

### Ems

> DO NOT use ems for `font-size`, because they are based on this value. You can use `ems` for the following:
> - padding
> - margin
> - element sizing

An `em` is relative to the element's `font-size`---its _local font size_---and its value varies depending on the element that that you're applying it to. 1 `em` means 'the font size of the current element'.

```css
.padded {
  font-size: 16px;
  padding: 1em;     /* computes to 16px */
  /* padding: 1.5em;   computes to 24px */
}
```

If you define the `font-size` with an `em`, then the size of the `em` is derived from the inherited font size. If you use `ems` to define other properties like `padding`, then the browser first calculates the font size, then calculates the `padding` with that font size. So, `font-size` and `padding` might have the same declared value but different computed values.

> To find the pixel size in `em` units, divide the desired pixel size by the inherited pixel size. For example, if you want a 20px font and the parent font is 16, 20/16 = 1.25 em.


## Rems

> You can use rems for font sizes.

Rem is short for "root em". A `rem` is relative to the root `em`. The root is the `html` tag, which is the root of the HTML document. You can access the `html` tag with the `:root` pseudo-class selector so you can have the specificity of a class element rather than a tag.

Because rems are relative to the root element, it has the same computed value throughout the stylesheet.



## Fonts

Set the `font-size` in the `:root` pseudo-class. Use `em` to set it here, and then `rem` throughout the stylesheet to change the size, relative to the `:root` setting.

The following example sets the inherited font size to 14px:

```css
:root {
  font-size: 0.875em;
}
```
> For most browsers, the default font size is 16px.

## Relative values

- A CSS pixel =/= to a monitor pixel
- **Computed value**: Absolute value that the browser computes for values declared using relative units
- Whe an element has a value defined using a length (px, rem, em, etc), its computed value is inherited by its child elements.

When to use rems:
- font sizing. Always use relative units when setting a font size because when users alter the screen zoom with + or -, pixels do not resize correctly.

When to use ems:
- padding
- margins
- element sizing
- border-radius

pixels:
- borders

percentages:
- container widths (as necessary)
