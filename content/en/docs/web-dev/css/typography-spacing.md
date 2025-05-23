---
title: "Typography and spacing"
linkTitle: "Typography"
weight: 150
# description:
---

## Text and typography

Good article: [Modern CSS Techniques to Improve Legibility](https://www.smashingmagazine.com/2020/07/css-techniques-legibility/)

If your `font-family` property setting uses this hierarchy:

1. First setting
2. Fallback font
3. Default user agent style

### System font stack

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
### ch unit

To ensure each line of content does not extend beyond a readable size, you can use the `ch` unit to set the `max-width` of a content block. A `ch` unit is as wide as the `0` (zero) character of the current font size:

```css
body {
  ...
  max-width: 78ch;
  margin: 0 auto;
  font-family: "Times New Roman", Times, serif;
  border-left: 5px double rgba(0, 0, 0, 0.16);
  min-height: 100vh;
}
```

## Numbers

Fonts might use either old-style or modern numbers. Old-style numbers can display above the meanline (top of the font line) and below the baseline of the text. Modern numbers display within the meanline and baseline. Control this and other number features with `font-variant-numeric`. 

### font-variant-numeric

This property lets you control the following:
- whether zeros have a slash
- number alignment
- how fractions are displayed

To change old-style number alignment to modern, apply this ruleset:

```css
body {
  font-family: "Raleway", sans-serif;
  color: #171717;
  font-variant-numeric: lining-nums;
}
```



## ems vs px

Use ems or rems when you think that spacing should resize if the user has a different font setting.
- ems might be the better choice for small measurements that immediately surround text or buttons
- px is fine if gaps or spaces do not need to scale for larger fonts

When to use margins vs padding:
- padding if you need to apply spacing to an entire container
- margin when you need to apply spacing to the outside of elements and there is a background color or image

## Line height

In the box model, the elements content box is surrounded by padding, then border, then margin, but the **height of the content box is determined by the font size and line height**:
- line height is usually defined in the body and inherited
- multiply the line height by the font size, and that is the total line height + font size. The measurement uses the same unit as the font size
- line height is distributed evenly above and below the text
- Ex: line height is 1.4, and font size is 16px, then the total height of the content box is 1.4 x 16 = **22.4px**. This means that there is an additonal 6.4px of space in the content box, with 3.2px on top, and 3.2px on the bottom.
  - Line height complicates margin calculations. If you give 10px of margin, then you have 13.2px margin due to line height

The fix is to subtract from the margin the extra space provided by line height. So, subtract the line height from the top element and the bottom element:
- `leading-trim` is an upcoming property that will remove the extra space from the content box

### Inline elements

The height of an inline element is determined by line height only. You can't change the height with padding.
- If you have to add distance between inline elements that might wrap, you have to set the line height.

This example updates the line height on an inline list of metadata tags. Because they are inline, their height has to be set with `line-height`. The line height set on the body is `1.4`. If you try to change the height with padding, the padding will overlap when the elements wrap on the line. You can achieve this with flexbox, but it requires that you add horizontal and vertical margins:

```css

@layer modules {
  .tag-list {
    list-style: none;
    padding-inline-start: unset;
  }

  .tag-list > li {
    display: inline;                    /* inline elements */
    padding: 0.3rem 0.5rem;
    font-size: 0.8rem;
    border-radius: 0.2rem;
    background-color: var(--light-gray);
    line-height: 2.6;                   /* added line height */
  }
}
```

## Web fonts and local fonts

A _font_ is a variant or weight of a specific type of fonts. _Typeface_ is the entire family of fonts--all available font weights, sizes, etc.

Historically, web designers had to choose from a limited set of typefaces called _web-safe fonts_, like Arial, Helvetica, and Georgia. If you specified less-safe fonts, they would only show up for users that had those fonts installed on their systems.

You can use web fonts or local/self-hosted fonts. Web fonts are imported from a service like Google Fonts, either in the HTML with a `<link>` tag or with an `@import` statement in the stylesheet. Local or self-hosted fonts are stored locally on your machine and imported using `@font-face`.

Here are the advantages and disadvantages of each:

|            | Advantages                                  | Disadvantages                                                                   |
| ---------- | ------------------------------------------- | ------------------------------------------------------------------------------- |
| web font   | Easy imports<br>Cached on user machine      | Extra call to import file<br>Privacy concerns<br>Service might discontinue font |
| local font | No dependencies<br>More performance control | Manage performance<br>User won't have font cached                               |


### Adding web fonts

Web fonts tell the browser where to download the fonts for the web page. Easiest way to use fonts is through an online service:
- [Google Fonts](https://fonts.google.com/)
- [Font Library](https://fontlibrary.org/)
- [Adobe Fonts (not free)](https://fonts.adobe.com/)

You can include them with link tags in the HTML or `@import` statement at the top of the stylesheet.

Google serves fonts in the WOFF2 format--Web Open Font Format, which is a compressed format designed for use over a network.

After images, online fonts are the largest resource that a page loads. **Do not download all fonts for a typeface, only select the ones that you need**.

If you use a font that you do not have downloaded or linked, then the browser approximates the style. This can have unwanted consequences.

This adds [Google Fonts](https://fonts.google.com/):

1. After you select your fonts, add the `<link>` tag to your HTML:
   ```html
   <link rel="preconnect" href="https://fonts.googleapis.com">
   <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
   <link href="https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,300;1,300&family=Sansita:wght@800&display=swap" rel="stylesheet">
   ```
   The last link is a stylesheet that Google provides. You can view it in a browser.

2. Next, add the fonts to your elements with the `font-family` property:
   ```css
   @layer global {
        body {
          font-family: Roboto, sans-serif;
          line-height: 1.4;
          background-color: var(--extra-light-gray);
          color: var(--text-color);
        }
        ...
    }
    ```

### @font-face

**Add `@font-face` at the top of the stylesheet.**

Use `@font-face` for locally-stored, purchased fonts that aren't available at an online service. `@font-face` only makes the fonts avaialble to your CSS--you still have to apply it to elements.

If you follow the last link in the Google fonts `<link>` set, you see a page that lists all font faces available to your page. Here is a specific rule for italic Roboto with font weight of 300. Each variant that you select has its own rule:

```css
/* latin */
@font-face {
  font-family: 'Roboto';            /* What you can reference in your stylesheet */
  font-style: italic;
  font-weight: 300 700;
  font-stretch: 100%;
  font-display: swap;
  /* Font file location for download */
  src: url(https://fonts.gstatic.com/s/roboto/v47/KFOKCnqEu92Fr1Mu53ZEC9_Vu3r1gIhOszmOClHrs6ljXfMMLt_QuAj-kw.woff2) format('woff2');
  /* Google puts the most often used characters in a font file for performance optimization */
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
```

- `font-family`: Name we use to refer to our font
- `font-style`:
- `font-weight`: Can load multiple font weights
- `font-stretch`:
- `font-display`: How the font is loaded. Font loading blocks other resources. `swap` means that the browser will load the font for a period of time. If the font isn't loaded after that time is up, then it loads other resources and finishes loading the font later.
- `src`: Font file location for download. Accepts a comma-separated list of locations and formats that are loaded in order until success
  
Here are some links:

- [W3 general info](https://www.w3schools.com/css/css3_fonts.asp)
- [Font file formats](https://fileinfo.com/filetypes/font)

### Common file types

TLDR; Use WOFF or WOFF2

- **TrueType (TTF)**: Not compressed
- **Open Type (OTF)**: Evolution of TFF, more chars
- **Embedded Open Type (EOT)**: Developed by MS, only supported by IE
- **Web Open Font Format (WOFF)**: Recommended by W3C. Created for web, compressed, contains copyright metadata
- **Web Open Font Format 2 (WOFF2)**: Continuation of WOFF, 30% more compressed
- **SVG**: Created to allow embedded glyph information in SVGs before web fonts were widespread

### Custom @font-face

Here is a `@font-face` rule that you would add to your stylesheet. The `local()` function tells the browser to check the user's system before searching the network:

```css
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 300;
  src: local("Roboto Light"), local ("Roboto-light"),
       url(https://path/to/file) format('woff2');
}
```

### Italic fonts

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

### Variable fonts

Newer fonts are available as variable fonts, where you can use a single font file for a range of fonts. (Read page 348-49 for details).

Basically, web fonts for different weights or sizes are included in a single font file. Italics are generally in their own file.

Variable fonts are not widely supported. You should add them with a `@supports` at-rule so they are applied when supported. This example loads the fonts when they are supported and uses a `not` clause to load fallbacks:

```css
@supports (font-variation-settings: normal) {
  @font-face {
    font-family: "Open Sans";
    src: url("fonts/open-sans-variable.woff2") format("woff2-variations");
    font-style: normal;
    font-weight: 100 800;
    font-display: swap;
  }
}

@supports not (font-variation-settings: normal) {
  @font-face {
    font-family: "Open Sans";
    src: local("Open Sans Regular"), local("OpenSans-Regular"),
      url("fonts/open-sans-regular.woff2") format("woff2"),
      url("fonts/open-sans-regular.woff") format("woff");
    font-weight: normal;
    font-display: swap;
  }

  @font-face {
    font-family: "Open Sans";
    src: local("Open SansBold"), local("OpenSans-Bold"),
      url("fonts/open-sans-regular.woff2") format("woff2"),
      url("fonts/open-sans-regular.woff") format("woff");
    font-weight: bold;
    font-display: swap;
  }
}
```

## Performance

As the browser downloads font files, the page might be ready to render. In some scenarios--if a fallback font is installed on the user's system--another font might render first, and then the web font renders when it is downloaded. This causes a shift in the page layout, which is called **FOUT** (Flash of Unstyled Text).

To avoid this, browsers first render the layout but do not render the text. This is called **FOIT**, Flash of Invisible Text.

`font-display` is how you manage font loading. It goes inside the `@font-face` rule:
- `swap`: Enables FOUT, where the fallback is displayed immediately and then the primary font displays when loaded.
- `auto`: Default browser behavior
- `block`: Enables FOIT, whih displays invisible text when loading. After a timeout period, the browser displays unstyled text until it is replaced with the primary web font.
- `fallback`: Compromise between `block` and `swap`. Text is invisible, then the fallback is used, then the web font is used.
- `optional`: Similar to `fallback`, but lets the fallback font stay in place if the web font doesn't load. Slower connections may never download the web font.

The Google Fonts URL has a `&display=swap` query parameter that you can edit to use any of these values.

Recommended usage:

- `fallback` for fast connections
- `swap` for slow connections
- `optional` when web font is not important to the design

### Testing font-loading

You can artificially slow down font-loading speeds with your dev tools:

1. In Chrome dev tools, select the **Network** tab.
2. Select **Disable cache**.
3. In the dropdown beside **Disable cache**, select the speed you want to test. For example, **Regular 3G**.


## Spacing

Text spacing involves adjusting these properties:
- `line-height`: Vertical space between lines of text
- `letter-spacing`: Horizontal space between individual characters

Typographical terminology includes leading and tracking:
- leading rhymes with bedding, and is equal to the font size plus the distance between the next line. Its the same as line height, but expressed in `pt` units.
  To convert `pt` to line height, multiply the value by 1.333, then divide by your font size to find the unitless line height.
- tracking is the space between characters and is usually expressed in a number such as 100, which represents thousandths of an em: 100 / 1000 = 0.1em.

### line-height

To find the right values, test using values that are too tight and then too far apart and select the one that looks best. There are some general rules of thumb to help:
- `line-height` is 1.2 by default. **This is usually too small--aim for 1.4 to 1.6**. Set this on the `<body>` element.
- Unitless `line-height` values are computed relative to the font size of the element. Use unitless values!
- Setting `line-height` to 1.2 is equivalent to setting it to `120%`.
- Longer lines of text should have larger line height for readability
- Lines should be **no longer than 45 - 75 characters**

### letter-spacing

Usually, the default letter spacing is fine, but it might require some tweaks, including on buttons and headings:
- `letter-spacing` takes a length, which is the distance between each letter
- Use very small values, 1px is too much. Use increments, starting with `0.01em`.
- body text and heading text should use different spacing.
  - Always make your headings too long to test the vertical line wrap spacing

```css
@layer global {
  body {
    font-family: Roboto, sans-serif;
    line-height: 1.4;
    letter-spacing: 0.01em;
    ...
  }
  ...
}
```

### Justified text

Apply `text-align: justify;` to justify text:
- `text-justify` property to determine how space is distributed between words. When omitted, the browser determines the space.
- `hyphens: auto;` tells the browser to add hyphens at the end of the line. This will help reduce the amount of whitespace between words.

```css
p {
  text-align: justify;
  hyphens: auto;
}
```
### Drop caps

A drop cap is a large, decorative capital letter used as the first letter in a paragraph or section. This example targets the letter with pseudo-classes and uses `float` to place first letter on the left and let the text wrap around it. It also sets the `line-height` low because it is calculated using the larger font size:

```css
p:first-of-type::first-letter {
  font-size: 6em;
  float: left;
  line-height: 0.5;
  font-family: "Passions Conflict", cursive;
}
```


### Headings, small elements, and spacing

Body text and heading text should use different spacing.
- Always make your headings too long to test the vertical line wrap spacing
- Headings can use negative spacing, which is useful for short spans of text

```css
.hero h2 {
    margin-block: 0 10px;
    font-size: 1.95rem;
    letter-spacing: -0.02em;
  }
.top-nav a {
    font-size: 0.8rem;
    ...
    padding: 0.3rem 1.25rem;
    ...
    text-transform: uppercase;
    letter-spacing: 0.03em;
  }
```