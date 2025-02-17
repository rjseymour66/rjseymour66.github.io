---
title: "Relative Units"
# linkTitle: "CSS in Depth"
weight: 30
# description:
---

## Definitions

Absolute units
: Definite - they do not change in context:
  - Absoulte units:
    - `px`: pixel - difficult to measure now with high-res screens. `96px` is about 1 physical inch on the screen.
    - `mm`: millimeter
    - `cm`: centimeter
    - `Q`: quarter-millimeter
    - `in`: inch 
    - `pt`: point 
    - `pc`: pica
  - `12pt` is equal to `16px`


Computed value
: Absolute value that the browser computes for values declared using relative units. When an element has a value defined using a length (px, rem, em, etc), its computed value is inherited by its child elements.

Length
: Formal name for a CSS element that has a unit, which is formally called a distance measurement.


Relative units
: Different values depending on the context they are called in
  - They are evaluated by the browser to an absolute value
  - rems and ems are the most common

Responsive design
: When styles respond differently based on the size of the browser window.

## Suggested units

[Youtube link](https://www.youtube.com/watch?v=N5wpD9Ov_To)


| Unit | CSS property | 
|:---|:---|
| `rem` | `font-size`<br>document flow (consistent spacing)<br>`padding`<br>`margin` |
| `em` | media queries<br>document flow (with more space) |
| `px` | `shadow-box`<br><br>`border`<br>`border-radius` |
| `%` | page/container widths |

- `height`: use min-height() when you need to set a height so that the content does not overflow at the bottom if the viewport size changes.

## Font size

Set the `font-size` in the `:root` pseudo-class. Use `em` to set the font size with the `:root` pseudo-class, and then `rem` throughout the stylesheet to change the size, relative to the `:root` setting.

The most top-level element in the DOM is the `<html>` element. You can select it with `:root`, a special pseudo-class selector:
- rems are relative to the root element, not the current element
- this makes it consistent across the document

Set the root element font-size to `1em` to set it equal to the browsers default font-size of 16px:
- usability tools resize fonts with relative units, no absolute - use relative to define font size

The following example sets the inherited font size to 14px:

```css
:root {
  font-size: 0.875em;
}
```

If you change the font size for an element with rems, you do not have to change the padding or other relative properties because their values are calculated according to the local `font-size` value.


### Scaling font sizes smoothly

If your font sizes change between screen sizes, they can change drastically at your breakpoints. Address this with the `clamp()` or `calc()` functions.

Setting a responsive font size means that other elements on the page scale appropriately, and without breakpoints.

#### clamp()

Accepts a minimum, preferred, and maximum value:
- the preferred value must be a calculation or it just applies the preferred value across all viewport sizes
 
```css
clamp(min-val, preferred-val-expression, max-val)

:root {
    font-size: clamp(0.9rem, 0.6rem + 1svw, 1.5rem)
}
```

#### calc()

> When working with font sizes, prefer `clamp()`.

Perform basic arithmetic with two or more values:
- good when you want to use values with different units
- when calculating font size, always use one unit with em or rem


This ruleset uses `0.5em` as the minimum font size, and it scales by `1svw` as you change the screen size:
```css
:root {
    font-size: calc(0.5em + 1svw);
}
```

## Relative units

- `rem` for properties that you _do not_ want to scale.
- `em` for properties that you _want_ to scale.

Styling with relative units is helpful because you do not know the absolute size of the screen that your document will display on - you don't know exactly how to size your elements
- Ex: Set a font size with relative units so it scales proportionately with the size of the window
- Ex: Set size of everything relative to the base font size and then resize everything by changing the base font size only


### ems

Ems are the most common relative length unit, comes from typography.

Uses the local font size to determine its _computed value_:
- the computed value of a font size is derived from its inherited value
- if font size is `1.5em`, then the size of `1em` for that element is `24px`. (16px * 1.2 = 24px)
  - If `font-size` is `16px` and the padding is `2em`, then the padding is `32px`
- tricky for nested elements that use ems for font-size and other properties, like padding
  - If em size is smaller than 1, child elements computed value is smaller (0.8em * 0.8em = 0.64em)
  - If em size is larger than 1, child elements computed value is larger (1.5em * 1.5em = 2.25em)

> DO NOT use ems for `font-size`. If you define the `font-size` with an `em`, then the size of the `em` is derived from the inherited font size. If you use `ems` to define other properties like `padding`, then the browser first calculates the font size, then calculates the `padding` with that font size. So, `font-size` and `padding` might have the same declared value but different computed values.


```css
.padded {
  font-size: 16px;
  padding: 1em;     /* computes to 16px */
/*padding: 1.5em;   computes to 24px */
}
```

### rems

Use `rem` for font sizes.

Rem is short for "root em", so a `rem` is relative to the `font-size` value in the root. The root is the `html` tag, which is the root of the HTML document. You can access the `html` tag with the `:root` pseudo-class selector so you can have the specificity of a class element rather than a tag.

Because rems are relative to the root element, it has the same computed value throughout the stylesheet.

[This article](https://codyloyd.com/2021/css-units/) says to use rem for fonts and px for everything else. This is because padding and margin scale along with the font. You might want to use rem instead of px at times.


## Viewport-relative units

Examples and implementations in this CSS tricks article, [Fun With Viewport Units](https://css-tricks.com/fun-viewport-units/).

Use cases:
- full-height heroes
- full-screen app-like interfaces
- Responsive typography
- Full-Height layouts, hero images, and sticky footers
- Fluid aspect ratios
- [Full width containers in Limited width parents](https://css-tricks.com/full-width-containers-limited-width-parents/)
- Scroll indicators

The _viewport_ is the area in the browser window where the web page is displayed - NOT including the address bar, shortcuts, toolbars, etc. The following are the basic units:

- `vh`: 1% of the viewport height
- `vw`: 1% of the viewport width
- `vmin`: 1% of the smaller dimension, height or width
- `vmax`: 1% of the larger dimension, height or width

So, `25vh` is 25% of the viewport's height.

> Viewport units are good for large hero images. However, this is an issue with mobile devices because some mobile devices have a feature that hides the address or nav bar to save space. This causes viewport-relative items to change size on the screen. This is called _layout thrashing_.
>
> Also, remember that viewports do not take scrollbars into account.

### vmin

`vmin` lets you set the height or width depending on the size of the viewport. This example sets the height to 90vh if the screen height is smaller than the width, and sets it to 90vw if the width is smaller than the height:

```css
.div {
    width: 90vmin;
    height: 90vmin;
    background-color: #369;
}
```

### Large and Small units

Viewport units were used for heroes, but some mobile devices dynamically hide menus when you are not scrolling and then display them when you are scrolling. This can cause the content on the screen to jump if you use viewport units. This is called _layout thrashing_.

This is largely resolved in the mobile browsers, but it might still be an issue.

To address this, CSS came up with large and small viewports:
- large is when all UX components are hidden. Prepend an `l` to the unit (`lvh`)
- small is when UX components are visible. Prepend an `s` to the unit (`svh`)
  - When in doubt, use small viewport units

This is not used very often.

When a viewport size changes because the device hides a UX component, you can use the large (for screens without the address bar):

- `lvw`
- `lvh`
- `lvmin`
- `lvmax`

Or small (for screens with the address bar):
- `svw`
- `svh`
- `svmin`
- `svmax`

## line-height

Some properties accept unitless values, but you can also use 0 without a unit, because 0 is equal to 0 of any length.

Accepts both units and unitless values, but prefer unitless because they are inherited differently:
- Unitless values are recalculated for all child elements
  - Child elements inherit computed values if the value is assigned with lengths (units)
  - This causes issues with line-height


To calculate the line-height, multiply the `font-size` by the `line-height` value:
  - Ex: `16px` (`1rem`) * `1.1` = `17.6px` 

```css
/* total line height is 38.4px */
body {
    line-height: 1.2;
}

.about-us {
    font-size: 2em;
}
```

## Custom properties (variables)

Create dynamic and context-based styling:
- declare a variable and assign a value to reference throughout your stylesheet
- more versatile than preprocessor variables
- create a variable with double hyphens (`--var-name`) and assign with `var()`
  - `var()` accepts a second optional fallback value
- Set global vars in the `:root` selector so they're available for the entire page
- These vars have scope - you can redifine variables within rulesets

```css
:root {
    --main-font: Helvetica, Arial, sans-serif;
    --brand-color: #369;
}

p {
    font-family: var(--main-font, sans-serif);
    color: var(--brand-color, blue);
}

.dark {
    margin-top: 2em;
    padding: 1em;
    background-color: #999;
    --main-bg: #333;            /* redefine global vars */
    --main-color: #fff;
}
```