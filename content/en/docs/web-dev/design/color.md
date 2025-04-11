---
title: "Color theory"
linkTitle: "Color"
weight: 80
# description:
---

Color theory is the science and art of applying color, how we perceive it, and the visual effects created when colors are mixed and matched:
- Always refer to the color wheel when you are selecting colors for your project
- Create different colors based on the primary colors

## Contrast checker

[WebAIM contrast checker](https://webaim.org/resources/contrastchecker/)

## Terminology

Shades
: Adds black to darken the color.

Tint
: Adds white to lighten the color.

Tone
: Add gray to dull the color.

Hue
: Another name for the pure color that does not have any white, black, or gray added.

Saturation
: How vivid and intense a color is. On scale 1-100, 100 is the full hue.

Lightness
: How light or dark a color is, or how much white or black it contains

### Creating shades, tints, tones

Start with the primary or main color that you want to create an alternate color for:
- Change brightness slider to add black and create a shade
- Change saturation to add white and create a tint

### Warm and cool colors

Warm colors: red, orange, yellow
- Happier, energetic
- attention-grabbing
Cool colors: green, blue, purple
- calm and relaxed
- recessed, appear further back

High saturation creates contrast, but too much creates clashing colors.

## Color wheel

Helps you ID color relationships and which ones work best together:
- Primary colors: red, yellow, blue
- Secondary colors: Created by mixing two primary colors. Ex: red + blue = purple
- Tertiary colors: Mix two colors that are adjacent to each other. In-between colors like red-orange.

### Monochromatic color scheme

A color theme that uses various shades and hues of one color:
- helps simplify busy designs

### Complementary color scheme

Complimentary colors are exact opposites on the color wheel:
- creates a bold color scheme
- the amount of contrast might make it hard to work with
- should decide which is primary, and which is the accent, then mix in a neutral color like black, white, or gray
- Helps focus users
- Show contrast between different states or values

### Analogous color scheme

Three colors on the color wheel that sit next to each other:
- blends colors harmoniously and is less intense
- occur in nature, like the colors in a sunset
- low contrast
- use on UI elements that don't overlap

### Triads color scheme

Colors that are evenly spaced around the color wheel:
- Ex: primary colors
- 60:30:10 interior design color rule using triads: use primary for 60%, secondary 30%, and accent color 10%
- use shades and tints to bring down color intensity, as needed

### Split complementary color scheme

Select a color on the color wheel, then select colors that are adjacent to its complementary

## Color psychology

Carl Jung pioneered this. Here are some quick tips and notes:
- **Red** attracts attention. It symbolizes strength, dominance, power, war, danger, sexuality, love, and passion.
- **Orange** indicates danger or warning and attracts attention. I symbolizes creativity, amusement, adventurous, energy, and activity.
- **Yellow** indicates caution or warning. It symbolizes positivity, optimism, amusement, and happiness.
- **Green** is calming. It symbolizes growth, nature, safety, healing, relaxation, and freshness.
- **Blue** is neutral and calming. It symbolizes tranquility, calm, trust, authority, and loyalty.
- **Purple** is associated with power and luxury. It symbolizes ambiguity, mystery, individualism, and spirituality.
- **Black** is a good neutral color that can be elegant and black. It can also symbolize darkness, power, sophistication, elegance, mourning, and evil.

## Picking and applying a color scheme

1. Always pick one color that is your base color and one or two accent colors
   - From these colors, create several tints and shades
   - Apply 60% saturation and 20% saturation
2. Pick a few shades of gray
   - Can have a hint of primary color
   - Keep the hue the same, but reduce saturation on primary color
   1. reduce brightness to 45%
   2. Create colors with 20% saturation, 15%, and 10%
   3. For last gray, saturation 5% and brightness to 60%
3. Apply to your UI. Can use the 60:30:10 rule 


## Web color modes

- Subtractive: CMYK--cyan, magenta, yellow, and _key_ for black.
- RGB and RGBA: red, blue, and green, and alpha. alpha is the opacity
- Hex: Maps RBG colors to hexadecimal #RRGGBB. The eight-digit version can set opacity, but its not widely supported on mobile browsers
- HSL and HSLA: hue, saturation, lightness, and alpha. Intuitive way to modify colors without referencing a color tool. You just set your hue, then change the saturation and lightness to create shades and tints.
  - hue: Number from 0-360, where each number is a degree on the color wheel
  - saturation: Percent from 0%-100%
  - alpha: opacity, accepts number from 0.0 to 1.0.
- CIE Lab and LCH: device-dependant color space defined by the International Commission on Illumination (CIE). "L" stands for lightness.
  LCH is lightness, chroma, and hue. Chroma is the amount of color you want to display.
