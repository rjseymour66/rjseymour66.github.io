---
title: "Relative Units"
# linkTitle: "CSS in Depth"
# weight: 20
# description:
---

_Relative units_ have different values depending on the context they are called in
- They are evaluated by the browser to an absolute value
- rems and ems are the most common

Length
: Formal name for a CSS element that has a unit, which is formally called a distance measurement.

_Absolute units_ are definite - they do not change in context:
- Absoulte units:
  - `px`: pixel - difficult to measure now with high-res screens. `96px` is about 1 physical inch on the screen.
  - `mm`: millimeter
  - `cm`: centimeter
  - `Q`: quarter-millimeter
  - `in`: inch 
  - `pt`: point 
  - `pc`: pica
- `12pt` is equal to `16px`

## Power of relative units

- Styling with relative units is helpful because you do not know the absolute size of the screen that your document will display on - you don't know exactly how to size your elements
  - Ex: Set a font size with relative units so it scales proportionately with the size of the window
  - Ex: Set size of everything relative to the base font size and then resize everything by changing the base font size only

## ems

Ems are the most common relative length unit, comes from typography
- ems are relative to the current element's font size
- Good for these elements because they scale evenly if the element inherits different font-sizes or the user changes the font settings:
  - `padding`
  - `height`
  - `width`
  - `border-radius`

### Font size 

Uses the local font size to determine its _computed value_:
- the computed value of a font size is derived from its inherited value
- if font size is `1.5em`, then the size of `1em` for that element is `24px`. (16px * 1.2 = 24px)
  - If `font-size` is `16px` and the padding is `2em`, then the padding is `32px`
- tricky for nested elements that use ems for font-size and other properties, like padding
  - If em size is smaller than 1, child elements computed value is smaller (0.8em * 0.8em = 0.64em)
  - If em size is larger than 1, child elements computed value is larger (1.5em * 1.5em = 2.25em)


## rems

Stands for 'root em'. Use rems for:
- font-size

The most top-level element in the DOM is the `<html>` element. You can seelct ti with `:root`, a special pseudo-class selector:
- rems are relative to the root element, not the current element
- this makes it consistent across the document

Set the root element font-size to `1em` - equal to the browsers default font-size of 16px:
- usability tools resize fonts with relative units, no absolute - use relative to define font size

```css
:root {
    font-size: 1em;
}
```