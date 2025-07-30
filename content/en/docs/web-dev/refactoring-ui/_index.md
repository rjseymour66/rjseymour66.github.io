---
title: "Refactoring UI"
# linkTitle: ""
weight: 40
description: >
  Notes on web design.
---

## Starting from scratch

- Don't try to design the app from the get go. Start with a feature and build it, then build another feature,...
  - Start building things as soon as possible so your imagination doesn't have to do all the heavy lifting
  - Expect each feature to be hard to build, and design the smallest, most useful version
- Try using a sharpie and paper for the first design. This prevents you from focusing on low-level details like typefaces
- Design in grayscale so you don't focus too much on color--you focus on space, contrast, and size.
- Fonts: Serifs are elegant and formal, sans-serif is playful
- Border radius: Small border radius is more formal, larger is less formal
- Color: Blue is neutral, pink is playful, gold is sophisticated
- Process of elimination: take a guess at what looks best, then try the option above and below that. This requires you have systems already designed.

### Systemize everything

Define systems in advance so you don't spend tons of time deciding what to use:

- Font size
- Font weight
- Line height
- Color
- Margin
- Padding
- Width
- Height
- Box shadows
- Border radius
- Border width
- Opacity

## Heirarchy is eveything

Visual heirarchy is how important the elements in an interface appear in relation to one another. It's the most effective tool you have to make something look "designed".

## Layout and spacing

- Always start with too much white space (margin and padding) and remove it until you're happy.
- Create a spacing system so you don't spend too much time nitpicking between values.

## Scaling layouts

- Use fixed widths--not percentages--for elements that should be fixed width like sidebars. Grid layouts don't work on elements like sidebars because they scale relative to the screen size.
- Give elements a max-wdith, and force them to shrink when the screen is smaller than the contents.


Sizing relationships do not have to be a defined ratio. Remember that elements that are large on large screens need to shrink faster than elements that are already small. For example, a heading is large on a large screen, but it should be just a bit larger than the body text on a small screen. The body text should be just a bit smaller on the small screen than the large screen. So, the ratio between the element sizes is not equal between screen sizes.

For padding, consider the size of the element. With a button, you want more padding at larger sizes and less at a smaller size. This means that maybe the padding doesn't size with the font size--use `rem` instead of `em`.

### Spacing and sizing system

See https://utopia.fyi/space/calculator/

Don't think about things as multiples of each other. For example, don't make everything a multiple of 4px. Think about the relative difference between adjacent values.

When you are dealing with small values, jumping from 4px to 8px is a big difference. For large values like a layout container, a 4px (or 4%) difference is not noticeable.

To build the system, start with a base value and build a scale using multiples of that value. A good base is 16px (1rem or 1em) because it is the default font size in all browsers. Make sure no two values in the scale are closer than about 25%. In other words, create smaller values with the base value, down to 16 x 0.25, and then larger values using any scale that makes sense. For example:

```scss
$padding-025: 0.25rem;
$padding-050: 0.5rem;
$padding-075: 0.75rem;
$padding-base: 1rem;
$padding-100: 1.5rem;
$padding-200: 2rem;
$padding-300: 3rem;
$padding-400: 4rem;
$padding-600: 6rem;
$padding-800: 8rem;
$padding-1200: 12rem;
$padding-2400: 24rem;
$padding-3200: 32rem;
$padding-4000: 40rem;
$padding-4800: 48rem;
```

When you space elements, make sure to consider proximity so related elements are grouped together. Do not create ambiguity by spacing elements incorrectly. For example:
- There should be more margin above a heading and less below so the reader knows that the heading is related to the text underneath.
- For unordered lists, make the space between bullets greater than the line height of each bullet.


## Text

### Type scale

Helpful links:
- fluid typography
- https://www.fluid-type-scale.com/
- https://typescale.com/
- https://utopia.fyi/type/calculator/

Creating a type scale is different than a spacing scale. Picking a base and then picking a ratio to multiple or divide that base by is called a _modular scale_. The issue with a modular scale include the following:
- It doesn't create enough values
- There are fractional sizes

A modular scale might work well for an article (or documentation...), but the solution for UI design is _hand-crafted scales_, where you just select values to choose from:

```scss
// need better naming...
$fs-12: 12px;
$fs-14: 14px;
$fs-base: 16px;
$fs-18: 18px;
$fs-20: 20px;
$fs-24: 24px;
$fs-30: 30px;
$fs-36: 36px;
$fs-48: 48px;
$fs-60: 60px;
$fs-72: 72px;
```

### Fonts

If you don't know what you're doing, use the system font stack:

```css
-apple-system, Segoe UI, Roboto, Noto Sans, Ubuntu, Cantarell, Helvetica Neue;
```

Otherwise, follow these rules:
- Pick a neutral sans-serif
- Ignore typefaces with less than 5 weights. They are usually not as well-crafted and are less popular. On Google fonts, filter your search with **Number of styles** and select **10+**.
- Sort by popularity if you really don't know

Consider letter spacing and x-height. The x-height is the size of the lowercase letter:
- Smaller text should have higher x-height and wider letter-spacing
- Fonts for larger text like headings have a shorter x-height and tighter letter spacing

### Line length

Paragraphs should be 45-75 characters wide. Use the `ch` unit for exact characters, or use 20-35ems.

### Alignment

Always align by baseline, not the center.

### Line-height

- Line-height should be proportional to your line length. Shorter lines can use 1.5, but longer lines might need close to 2.
- Small text needs more line height
- Larger text can use much less, closer to 1

### Links

If the link is not part of the main user path, maybe you don't need special styles on them. Instead, you could use only an underline or change the color only on hover.


### Numbers

Right-align numbers in a table so users can compare them by decimal place.

### Letter spacing

If you selected a popular typeface, you should trust its designer and leave the letter-spacing alone.

Here are scenarios where you might consider changeing the letter spacing:
- Headings for fonts optimized for small sizes. Adjust the letter-spacing to something like `-0.05em;`. This strategy works only for small text fonts. In other words, you cannot use heading style fonts (ex: Oswald) and increase the spacing.

Lowercase letters have visual variety and are easily distinguished from each other. They have ascenders and descenders that go above the x-height and below the baseline, respectively. Uppercase letters are not as distinguishable. Increase letter spacing when you use all caps to help with this. Start with something like `0.05em`.


## Color

Use HSL. The browser only understands HSL.
- HSB is common in design software
- Hex and RGB

HSL represents colors using attributes that the human-eye can intuitively perceive:
- hue: position on the color wheel. There are 360 degrees, where 0 and 360 are equal. 0 is red, 120 is green, 240 is blue
- saturation: how colorful or vivid the color looks. 0% is grey (no color) and 100% is vibrant and intense. Hue needs saturation, or the color is grey regardless of its degree on the color wheel.
- lightness: how close a color is to black or white. 0% is pure black, 50% is the pure color at the given hue, and 100% is pure white.

### Categories

You always need more colors than you think. Your color palette should include colors from three categories: greys, primary colors, and accent colors.

#### Greys

8-10 shades.

Use greys for text, backgrounds, panels, form controls, etc. Stay away from true black--start with dark grey and work up to white.

#### Primary colors

5-10 shades.

Primary colors determine the overall look of the site. Use for primary actions, active navigation elements. Darker shades are good for text, very light shades are great for backgrounds and alerts.

#### Accent colors

Up to 10 different colors with 5-10 shades _each_.

Accent colors get the user's attention or emphasize states, like red for a destructive action, yellow for a warning, or green for something positive (think callouts).

### Defining shades

