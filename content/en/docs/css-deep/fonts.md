---
title: "Fonts"
# linkTitle: "CSS in Depth"
weight: 60
# description:
---


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


### Responsive font sizes

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


## Text and typography

Good article: [Modern CSS Techniques to Improve Legibility](https://www.smashingmagazine.com/2020/07/css-techniques-legibility/)

If your `font-family` property setting uses this hierarchy:

1. First setting
2. Fallback font
3. Default user agent style

You can make sure you get the font that you want with the [system font stack](https://css-tricks.com/snippets/css/system-font-stack/), which is a long list of fallback fonts.

With variables:

```css
:root {
  --system-ui: system-ui, "Segoe UI", Roboto, Helvetica, Arial, sans-serif,
    "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
}

.element {
  font-family: var(--system-ui);
}
```

Without variables:

```css
/* Define the "system" font family */
@font-face {
  font-family: system-ui;
  font-style: normal;
  font-weight: 300;
  src: local(".SFNSText-Light"), local(".HelveticaNeueDeskInterface-Light"),
    local(".LucidaGrandeUI"), local("Ubuntu Light"), local("Segoe UI Light"),
    local("Roboto-Light"), local("DroidSans"), local("Tahoma");
}

/* Now, let's apply it on an element */
body {
  font-family: "system-ui";
}
```

### Online fonts

To prevent long lists of font-family values, you can include an online font library:

- [Google Fonts](https://fonts.google.com/)
- [Font Library](https://fontlibrary.org/)
- [Adobe Fonts (not free)](https://fonts.adobe.com/)

You can include them with link tags in the HTML or `@import` statement at the top of the stylesheet.

### Downloaded fonts

A downloaded font is a font that you downloaded from the web. This requires that you use the `@font-face` rule, then use the `font-family` property:

```css
@font-face {
  font-family: my-cool-font;
  src: url(../fonts/the-font-file.woff);
}

h1 {
  font-family: my-cool-font, sans-serif;
}
```

Here are some links:

- [W3 general info](https://www.w3schools.com/css/css3_fonts.asp)
- [Font file formats](https://fileinfo.com/filetypes/font)

## Font styles

### Italics

- `font-style`:
  - If italics are for styling purposes, use `font-style: italic;`
  - If italics are for emphasis, use `<em>` tags
- `letter-spacing`: space between letters
- `line-height`: space between lines in wrapped text.
- `text-transform`: Changes the case of the text (ex: all UPPERCASE)
- `text-shadow`: Shadow around text. `text-shadow: offset-x | offset-y | blur-radius | color` [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow)
- `ellipsis`: Use this with other properties to put an ellipsis in place of overflowing text:
  ```css
  .overflowing {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  ```
  You need this because text will write outside of its container, by default. See [CSS Tricks](https://css-tricks.com/snippets/css/truncate-string-with-ellipsis/) for in-depth info.
