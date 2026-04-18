+++
title = 'Colors'
date = '2025-08-06T10:50:52-04:00'
weight = 10
draft = false
+++

Color is one of the primary tools for creating contrast in a design. Contrast draws the user's attention to what matters most on the page.

- For contrast to work, you must establish patterns and then break those patterns to highlight the important part of the page.
- Establish contrast with color, spacing, or size.
- Every page asks users to do something: read a story, submit information, or complete a task. Use contrast to guide them toward that action.

## Defining color

A color palette typically centers on one primary color from which all other colors derive. Assign these values to custom properties so you can reuse them throughout the stylesheet.

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

You often need colors that fall between those defined in the design mockup. Name raw colors with numbers that correspond to their lightness value. Then create a second set of descriptive variables that reference these raw colors and use them throughout the stylesheet:

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

The following tools help you explore and derive colors:

- [Color theory introduction](https://tallys.github.io/color-theory/): A visual guide to color relationships, harmony, and contrast.
- [OKLCH color picker](https://oklch.com/#70,0.1,154,100): An interactive tool for picking and converting OKLCH colors.

To find a color that works well with another, find its _complement_. The complement is the color on the opposite side of the color wheel:

1. For HSL or OKLCH colors, add or subtract 180 from the hue value. If the result is negative, add 360 to bring it into range.
2. Adjust the lightness and chroma as needed. Start from the original color's lightness and chroma values.

### Font color contrast

Check contrast with [Odd Contrast](https://www.oddcontrast.com/#p3__color(display-p3_0.0967_0.167_0.4494)__color(display-p3_0.951_0.675_0.7569)): paste in your background color, then paste in your text color as the foreground to view the results.

Use a dark gray for black, or a light gray for white. Most text on a screen is dark gray rather than pure black because pure black creates excessive contrast, which causes eye fatigue.

W3C's Web Content Accessibility Guidelines (WCAG) provide contrast ratio suggestions:

- **AA**: Minimum recommended contrast ratio for most content.
- **AAA**: Stricter, enhanced contrast ratio for higher accessibility compliance.

Large text is 24px or larger for regular font weight, or 18.667px for bold fonts. The ratio is lower for large text because it is easier to read.

## Gamuts

A _gamut_ is a range of color that indicates how many colors a device can display or a color notation can represent. Hex colors are a historically popular format for CSS colors, but they can represent colors only within a limited gamut.

Devices and monitors increasingly support wider color gamuts:

- Previously, most devices were limited to a gamut called Standard RGB (sRGB), which includes just a portion of colors that the human eye can see.
- Current devices support Display P3, which is a 25% wider gamut than sRGB.
- Some monitors support the newest gamut, Rec2020, which represents 75% of all visible color.

Media queries can detect color gamut support:

```css
@media (color-gamut: p3|srgb|rec2020) {...}
```

### Color spaces

A _color space_ is a particular arrangement of a color in a gamut. Picture a cylinder: the top is white, the bottom is black, saturation runs from the core to the outside edge, and hue (the color category, such as red or blue) runs around the circumference.

## Color notations

[High-definition CSS color guide (Chrome Developers)](https://developer.chrome.com/docs/css-ui/high-definition-css-color-guide)

When you use a notation that supports a wider gamut, the colors you specify may exceed what the hardware can display:

- The browser rounds out-of-range colors to the closest possible representation.
- These notations are designed to be perceptually linear, meaning that equal changes in lightness value produce equal perceived changes across all hues, regardless of how the human eye perceives them.
- LAB and LCH were superseded by OKLAB and OKLCH, but browser support for all four is identical.
- For all notations, you can use the word `none` in place of a `0` value.

When in doubt about browser compatibility, use HSL, but OKLCH is gaining support across browsers.

### RGB and hex

RGB and hex are limited to the sRGB gamut, and their notation is not intuitive because it was designed to be read by a computer. Hex is useful when you need a compact notation that you can copy between applications.

The `rgb()` function no longer requires commas, and you no longer need the `rgba()` function. Use the slash notation for transparency instead: `rgb(1 2 3 / 0.6)`.

Hex indicates red, green, and blue with two-digit hexadecimal values:

- `#80c090` is `80` red, `c0` green, and `90` blue (hex), which equals 128 red, 192 green, and 144 blue (decimal).
- New eight-digit hex notation represents transparency with the last two digits, where `00` is transparent and `FF` is opaque.
  - `#80c09088` is a semi-transparent version of `#80c090`.
- Hex can be shortened to three-digit notation, where each digit is doubled. So, `#fff` is short for `#ffffff`, and `#c90` is short for `#cc9900`. There is also a four-digit notation that expands to the eight-digit transparency notation.

The `rgb()` function represents hex values in decimal notation.

### HSL

HSL is limited to the sRGB gamut. HSL stands for Hue, Saturation, and Lightness, and is designed to be more human-readable than RGB.

The `hsl()` function no longer requires commas, and you no longer need the `hsla()` function. Use the slash notation for transparency instead: `hsl(1 2 3 / 0.6)`.

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

HWB is limited to the sRGB gamut. HWB stands for Hue, Whiteness, and Blackness:

```css
hwb(198deg 12.5% 20.4%)
```

Hue works the same as in HSL: it is a degree on the color wheel. Whiteness and blackness are percentages, where 100% is pure white or black. If both white and black add up to 100%, the color is pure gray, and the greater of the two values determines its darkness.

### LCH and OKLCH

LCH and OKLCH are cylindrical color spaces, similar to HSL, defined with Lightness, Chroma, and Hue:

```css
lch(58% 39.8 241.5deg)
oklch(64% 0.12 233deg)
```

Lightness
: Perceived lightness, value from 0% (black) to 100% (white) or decimal from `0` to `1.0`.

Chroma
: How vivid the color is, where 0 is pure gray.

Hue
: Angle around the color wheel:

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
- Select the color box without SHIFT to open the color picker.
  - The color picker has a thin white line to indicate available colors. Colors to the left are sRGB, and colors to the right are part of Display P3.
  - If you use a Display P3 color on an sRGB monitor, the browser will approximate the color.

To force states on an element (like `a:visited`):

1. Right-click and choose **Inspect** or **Inspect Element** from the context menu.
2. In the Elements pane, right-click the `a` tag and select **Force State** > `:active`.
