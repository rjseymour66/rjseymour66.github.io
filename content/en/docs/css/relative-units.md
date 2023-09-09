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
/*padding: 1.5em;   computes to 24px */
}
```

If you define the `font-size` with an `em`, then the size of the `em` is derived from the inherited font size. If you use `ems` to define other properties like `padding`, then the browser first calculates the font size, then calculates the `padding` with that font size. So, `font-size` and `padding` might have the same declared value but different computed values.

> To find the pixel size in `em` units, divide the desired pixel size by the inherited pixel size. For example, if you want a 20px font and the parent font is 16, 20/16 = 1.25 em.


## Rems

> You can use rems for font sizes.

Rem is short for "root em", so a `rem` is relative to the `font-size` value in the root. The root is the `html` tag, which is the root of the HTML document. You can access the `html` tag with the `:root` pseudo-class selector so you can have the specificity of a class element rather than a tag.

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

### Change padding and margin with fonts

If you change the font size for an element with rems, you do not have to change the padding or other relative properties because their values are calculated according to the local `font-size` value.

> Use `rem` for properties that you _do not_ want to scale.
> Use `em` for properties that you _want_ to scale.

### Media queries

A media query uses the `@media` rule to specify styles that are applied to certain screen sizes or media types. You can define the root font-size in each media query:

```css
:root {
  font-size: 0.85em;
}
 
@media (min-width: 800px) {
  :root {
    font-size: 1em;
  }
}
 
@media (min-width: 1200px) {
  :root {
    font-size: 1.15em;
  }
}
```

> `min-width` is _mobile-first_ responsive design.

## Viewport-relative units

The _viewport_ is the area in the browser window where the web page is displayed. The following are the basic units:

- vh—1% of the viewport height
- vw—1% of the viewport width
- vmin—1% of the smaller dimension, height or width
- vmax—1% of the larger dimension, height or width

So, `25vh` is 25% of the viewport's height.

> Viewport units are good for large hero images. However, this is an issue with mobile devices because some mobile devices have a feature that hides the address or nav bar to save space. This causes viewport-relative items to change size on the screen. This is called _layout thrashing_.
>
> Also, remember that viewports do not take scrollbars into account.

When a viewport size changes because the device hides the address bar, you can use the large (for screens without the address bar):

- lvw
- lvh
- lvmin
- lvmax

Or small (for screens with the address bar):
- svw
- svh
- svmin
- svmax

### Viewports and font-size

You can use the `calc()` function to do basic arithmetic with two or more values.



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
