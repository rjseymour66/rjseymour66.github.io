---
title: "Color and contrast"
linkTitle: "xColor"
weight: 140
# description:
---

In design, contrast is a means of drawing attention to something by making it stand out.

- For contrast to work, you must establish patterns and then break those patterns to highlight the important part of the page
- Establish contrast with color, spacing, or size
- You want to tell the user a story, collect information, or complete a task

## Defining color

A color palette typically has one primary color that everything else is based on

- assign these values to custom properties

Define the colors in the `theme` module:

```css
@layer theme {
  :root {
    --brand-green: #076448;
    --dark-green: #099268;
    --medium-green: #20c997;
    --text-color: #212529;
    --gray: #868e96;
    --light-gray: #f1f3f5;
    --extra-light-gray: #f8f9fa;
    --white: #fff;
  }
}
```

### Naming color variables

You will often need colors that exist between those provided on the design, so name colors with numbers, where the numbers roughly correspond to the lightness in the color. Then, create a second set of values that use these colors but have descriptive names that you use throughout the stylesheet:

```css
/* raw color vars */
--gray-10: oklch(10% 0.01 165deg);
--gray-20: oklch(23% 0.01 165deg);
...

/* variables for stylesheet */
--background-4: var(--gray-10);
--background-3: var(--gray-20);
...
```

### Deriving new colors from your palette

- Great article on [color theory](https://tallys.github.io/color-theory/).
- [OKLCH color picker](https://oklch.com/#70,0.1,154,100)
-

To find a color that works well with another, find its _complement_. The complement is the color on the opposite side of the color wheel:

1. For HSL or OKLCH colors, add or subtract 180 to the hue value. Convert negative numbers to between 0 and 365.
2. Adjust the lightness and chroma as needed. Start with the original color's values that you used to find the complement.

### Font color contrast

Use an online tool to check contrast:

- [Odd Contrast](<https://www.oddcontrast.com/#p3__color(display-p3_0.0967_0.167_0.4494)__color(display-p3_0.951_0.675_0.7569)>). Paste in your background, and then paste in your text color as the foreground to view the results.

Use a dark gray for black, or a light gray for white. Most text on a screen is a dark gray rather than black because black creates too much contrast. Too much contrast can create fatigue.

W3C's Web Content Accessibility Guidelines (WCAG) provide contrast ration suggestions:

- AA: minimum recommendations
- AAA: stricter, enhanced content ratio

Large text is 24px or larger for regular font weight, or 18.667px for bold fonts. The ratio is lower for large text because it is easier to read.

## Gamuts

A _gamut_ is a range of color that is often used to indicate how many colors can be displayed on a device or represented by a particular color notation, such as hex.

- Hex colors are historically popular format for CSS colors, but they can represent colors only within a limited gamut

Devices and monitors keep improving and can represent more gamuts than ever before.

- Previously, most devices were limited to a gamut called Standard RGB (eRGB), which include just a portion of colors that the human eye can see
- Now, the gamut is Display p3, which is a 25% wider gamut than eRGB
- Some monitors support the newest gamut, Rec2020, which represents 75% of all visible color

Media queries can detect color gamut support:

```css
@media (color-gamut: p3|srgb|rec2020) {...}
```

### Color spaces

A _color space_ is a particular arrangement of a color in a gamut. Think of a cylindrical color space where the top is white, bottom is black, saturation is measured from the core to the outside, and hue (each individual color) is the circumference of the cylinder.

## Color notations

https://developer.chrome.com/docs/css-ui/high-definition-css-color-guide

Colors that represent the wider gamuts might display colors beyond the hardware's capability:

- The browser rounds to the closest possible representation
- These colors strive to be perceptually linear, meaning that lightness applies the same to all colors, regardless of how the human eye preceives it
- LAB and LCH were superseded by OKLAB and OKLCH, but browser support for all four is identical
- For all notations, you can use the word `none` in place of a `0` value

When in doubt about browser compatibility, use HSL, but OKLCH is gaining support across browsers.

### RGB and hex

RGB and hex are limited to the sRGB gamut, and their notation is not intuitive because it was designed to be read by a computer. Hex is great when you want a short notation that you can copy between applications.

`rgb()` function no longer requires commas. Also, you no longer need the `rgba()` function, because you can use the slash notation: `rgb(1 2 3 /0.6)`.

Hex indicates red, green, and blue with two-digit hexadecimal values:

- `#80c090` is `80` red, `c0` green, and `90` blue, and 128 red, 192 green, and 144 blue.
- New eight-digit hex notation represents transparency with last two digits, where `00` is transparent and `FF` is opaque.
  - `#80c09088` is a semi transparent version of `#80c090`
- Hex can be shortened to three-digit notation, where each digit is doubled. So, `#fff` is short for `ffffff`, and `#c90` is short for `#cc9900`. There is also a four-digit notation that expands to the eight-digit transparency notation

`rgb()` function represents hex values in decimal notation.

### HSL

Limited to the sRGB gamut.

`hsl()` function does not require commas any longer. Also, you no longer need the `hsla()` function, because you can use the slash notation: `hsl(1 2 3 /0.6)`.

Hue, Saturation, and Lightness - designed to be more human-readable.

Hue
: Angle between 0 and 359 that indicates the degrees around the color circle. You do not have to include the `deg` unit. Major degree transitions include:

- red (0)
- yellow (60)
- green (120)
- cyan (180)
- blue (240)
- magenta (300)
- red again.

Saturation
: Percentage that defines the color intensity. 100% is vivid color, 0% is a shade of gray.

Lightness
: Percentage that defines how light or dark the color is. 50% is vivid, 100% is pure white, 0% is black.

### HWB

Limited to the sRBG gamut.

Hue, Whiteness, Blackness:

```css
hwb(198deg 12.5% 20.4%)
```

Hue is the same as with HSL--it is a degree on the color wheel. Whiteness and blackness are percentages, where 100% is pure white or black. If both white and black add up to 100%, the color will be pure gray, and its darkness is determined by which of the two values is greater.

### LCH and OKLCH (Oklachroma)

Cylindrical color spaces like HSL and defined with Lightness, Chroma, and Hue:

```css
lch(58% 39.8 241.5deg)
oklch(64% 0.12 233deg)
```

Lightness: Perceived lightness, value from 0% (black) to 100% (white) or decimal from `0` to `1.0`.
Chroma: How vivid the color is, where 0 is pure gray.
Hue: Angle around the color wheel:

- red (0)
- yellow (90)
- green (140)
- cyan (195)
- blue (260)
- magenta (330)
- red again

## Dev tools

In Chrome, open dev tools and click on the `<html>` element to view your custom properties:

- Hold SHIFT and click on the small square of color to view other notations for the same color.
- Select the color box without shift to open the color picker.
  - The color picker has a thin white line to indicate available colors. Colors to the left are sRGB, and colors to the right are part of Display 3.
  - If you use a Display 3 color on an sRGB monitor, the browser will approximate the color.

To force states on an element (like `a:visited`):

1. Right-click and choose Inspect or Inspect Element from the context menu.
2. In the Elements pane, right-click the a tag and select Force State > :active