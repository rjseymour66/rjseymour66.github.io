---
title: "Typography"
# linkTitle: ""
weight: 70
# description:
---

Typeface sets the tone for the entire user experience.

## Basics

A typeface is a set of characters, and a font is a variation of that typeface in weight, size, etc. There are four categories of typefaces:
- serif: have small lines (serifs) at the end of larger strokes.
  - Professional, strict impression
- sans serif: sans = "without"--it comes from French. Have the same line thickness throughout the letter, can be hard to distinguish in big blocks of text and on smaller screens.
  - Modern, trendy, clean impression
- script: based on calligraphy and handwriting. Popular for wedding decorations. Good for headlines and titles. Don't use for body copy. Keep as large as possible for short lines of text.
  - Blogs, very informal
- decorative: Have ornaments, embellishments, or other treatments that make them unique. Use sparingly for headlines or section headings. DO NOT use for body text.


### Styles and weights

Italic and oblique:
- italics are slanted but seem more like cursive
  - `<em>`: when you want to add emphasis
  - `<i>`: distinguish text to create a mood for idiomatic, technical, or taxonmy
- oblique are normal typefaces but at a slant

### Weights

Create contrast in your typography and draw attention:
- all typefaces have at least normal and bold weights
- Use lighter font size for large text because lighter is hard to read on small text
  - `<b>`: draw attention to words
  - `<strong>`: these words have greater importance than the rest of the text

### Use 3 fonts
- body copy
- large headings
- smaller headings and text that needs to stand out
- **You can use one typeface, or one for body copy, and one for anything that isn't body copy**.

## Picking a font

Consider the following when selecting fonts:
- It should be readable and legible at the size you intend to use
- DO NOT use decorative fonts for long body copy. Use sans serif or serif
- Start with Google Fonts, then FontSquirrel. Google Fonts are free and optimized for the web

### Pairing typefaces and fonts

Create a cohesive set of typography that helps define the hierarchy of your page and draws readers' eyes:
- use different fonts of the same typeface family, like Roboto and Roboto Slab
- use different fonts of the same typeface by mixing and matching font weights

You can pair different typeface categories, like script and serif, serif and sans-serif, etc.

### Type ramp

A type ramp is a set of guidelines and font styles tht define the family, size, style, when and where fonts are used in your site:
- its your type strategy
- similar pages should share styles

Start here when defining your type ramp:
- Heading `<h1>`
- Subheading `<h2>`
- Title `<h3>`
- Subtitle `<h4>`
- Body `<p>`
- Caption `<caption>`

### Size selection

Body font should be defined as 1rem, with a base size between 16px and 18px. Then, create your type scale:
- Body: 1em
- Subtitle: 1.5rem
- Title: 2rem
- Subheading: 2.5rem
- Heading: 3rem

For more mathematical proportions, go to [Visual Type Scale Calculator](https://typescale.com/)


### Vertical rhythm (line height)

After you set the base font size, set the line height to set the spacing between all elements on your page. Guidelines and recommendations:
- WCAG version 2.1 says we need a minimum line-height value of 1.5em (or 150%) for main text
- Title should use 120-140% in case the lines wrap

The pixel value of your line height (font size in pixels x line height) is the basis of our vertical rhythm.
- use this to calculate the margins, padding, and line heights for different-sized fonts in type scale
- to start, set your top and bottom text margins to 24px - this is calculated from the line height


#### Calculate margins and padding for titles

Create vertical rhythms using multiples of your text line height:

1. start with 16px text and line height 1.5em (24px)
2. heading is 3rem, so its 48px. use line height of 1.25px so it is a round number, 60px
3. now, add margin and padding so the title line height is a multiple of our text line height
   - 72 is not enough, it only gives us 12px of margin to work with
4. 96px gives us 36px of margin to work with, so add 24px to the top, and the remaining 12px to the bottom margin

Now, there is logic to our margin and padding.

For body text, keep the space between paragraphs at the normal 1.5em line height to group them.

### Readability

- line length: Select between 45-70 characters wide, have also seen 40-80 wide
- letter spacing: do what looks normal