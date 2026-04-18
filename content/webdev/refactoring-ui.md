---
title: "Refactoring UI"
# linkTitle: ""
weight: 40
description: >
  A cheat sheet for UI design: spacing, typography, color, and visual hierarchy.
---

## Starting from scratch

Design one feature at a time, not the whole app at once. Build it, then move to the next. Starting
to build as soon as possible prevents you from relying entirely on your imagination. Expect each
feature to be difficult. Design the smallest, most useful version before expanding scope.

For early sketches, reach for a sharpie and paper. The resistance of the medium prevents you from
focusing on low-level details like typefaces and padding values. Design in grayscale first so you
focus on space, contrast, and size before committing to a color palette.

Every design communicates a *personality* before users read a word. Three properties drive most of
that signal:

- **Fonts:** Serifs read as elegant and formal. Sans-serif reads as playful and modern.
- **Border radius:** A small radius (2–4px) reads as formal. A large radius (12px+) reads as casual.
- **Color:** Blue reads as neutral and trustworthy. Pink reads as playful. Gold reads as sophisticated.

Real-world examples show how these signals combine:

| Trait | Formal | Playful |
|---|---|---|
| Border radius | 0–2px (Linear, Stripe) | 16px+ (Duolingo, Mailchimp) |
| Font | Serif like Freight Display (NYT, Goldman Sachs) | Rounded sans-serif like Nunito (Duolingo) |
| Color | Navy, charcoal, gold (Goldman Sachs) | Coral, bright green, teal (Duolingo, Headspace) |

When you are unsure which value looks best, make your best guess, then try the option immediately
above and below it on your scale. This approach only works when you have systems defined in advance.

### Systemize everything

Define systems in advance so you do not spend time deciding which value to apply each time you
design an element. A system is a constrained set of options. When you have ten spacing values
instead of unlimited pixels, decisions are fast and output is consistent.

Define systems for at least the following properties:

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
- Transition duration
- Z-index
- Breakpoints

Tailwind CSS's default configuration is a widely adopted reference for what a complete design
system looks like in practice. Reviewing it gives you a concrete anchor for each category above.

## Hierarchy is everything

*Visual hierarchy* is how important elements appear relative to one another. It is the most
effective tool for making an interface look designed rather than assembled.

Three properties create hierarchy: size, weight, and color. Most designers reach for size first,
but color and weight are often more powerful because they do not disrupt layout. On a Stripe invoice
page, the total amount appears at 48px bold while the invoice number sits at 12px in grey. The size
difference is large, but the weight and color do as much work as the size.

De-emphasizing secondary information is as important as emphasizing primary information. If
everything competes for attention, nothing wins.

You rarely need more than two font weights to create sufficient hierarchy. A regular and a semibold
cover most cases. GitHub's interface is a good example: navigation labels, issue titles, and body
text all live at similar sizes but are visually distinct because of weight and color alone.

Visual weight also comes from color saturation, icon size, and whitespace. A saturated element
draws the eye even at a small size. An element surrounded by whitespace appears more important than
an equally sized element that is crowded.

## Layout and spacing

Always start with too much whitespace (margin and padding) and remove it until you are happy. It
is far easier to tighten a spacious layout than to breathe air into a cramped one.

Notion's editor applies roughly 80px of horizontal padding on desktop. That feels generous at
first, but it keeps reading comfortable and gives the content room. Contrast this with early
Bootstrap grids, which defaulted to 15px gutters and made UIs feel dense.

Create a spacing system so you do not spend time choosing between arbitrary pixel values. See the
[Spacing and sizing system](#spacing-and-sizing-system) section for how to build one.

## Scaling layouts

For elements that should have a fixed width (like sidebars), set a fixed pixel value rather than a
percentage. Grid layouts cause sidebars to scale relative to the screen, which produces awkward
results at different breakpoints.

Give elements a `max-width` and allow them to shrink when the screen is narrower than the content.

Sizing relationships do not have to follow a defined ratio across breakpoints. Elements that are
large on large screens need to shrink faster than elements that are already small. For example, a
heading is large on a large screen but should be only slightly larger than body text on a small
screen. Body text should be only slightly smaller on a small screen than on a large one. The ratio
between element sizes is not constant across screen sizes.

For padding, consider the size of the element. A button needs more padding at larger sizes and less
at smaller sizes. This means padding should not scale with font size. Apply `rem` instead of `em`
to pin padding to the root font size rather than the element's own font size. For example, a button
with `padding: 0.5em 1em` collapses its padding when the button is set to a small font size.
Switching to `padding: 0.5rem 1rem` keeps the padding consistent regardless of the button's font
size.

Fluid typography scales element sizes smoothly between a minimum and maximum value as the viewport
grows. The [Utopia type calculator](https://utopia.fyi/type/calculator/) generates these values
automatically.

### Spacing and sizing system

See the [Utopia space calculator](https://utopia.fyi/space/calculator/) for a tool that generates
fluid spacing scales automatically.

The most important rule for building a spacing scale is to think about the relative difference
between adjacent values, not multiples of a base number. When dealing with small values, jumping
from 4px to 8px is a significant difference. For large values like a layout container, a 4px
difference is not noticeable.

To build the system, start with a base value and build a scale using multiples of that value. A
good base is 16px (1rem) because it is the default font size in all browsers. No two values in the
scale should be closer than about 25% apart. Build smaller values down from the base and larger
values up. The following variables show a complete scale:

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

When spacing elements, apply the principle of *proximity*: related elements should be closer
together than unrelated ones. There should be more margin above a heading and less below it so the
reader knows the heading belongs to the text underneath. For unordered lists, the space between
list items should be greater than the line height of each item.

A real-world example is VS Code's Settings panel, where each setting label and its control are 8px
apart, while the gap between separate settings is 24px. That spacing alone communicates what
belongs together before you read a word.

## Text

### Type scale

Helpful links:

- [Fluid type scale calculator](https://www.fluid-type-scale.com/)
- [Modular scale calculator](https://typescale.com/)
- [Utopia type calculator](https://utopia.fyi/type/calculator/)

A *modular scale* picks a base size and multiplies or divides by a fixed ratio to generate a type
scale. The problems with a modular scale include the following:

- It does not create enough values for UI work
- It produces fractional pixel sizes

A modular scale works well for articles or documentation, but the solution for UI design is a
*hand-crafted scale*, where you select values directly. The following example shows a complete
hand-crafted scale, which maps closely to Tailwind CSS's default font size scale:

```scss
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

If you are not confident in a typeface choice, apply the system font stack. It renders each
operating system's native UI font and requires no network request:

```css
system-ui, -apple-system, Segoe UI, Roboto, Noto Sans, Ubuntu, Cantarell, Helvetica Neue;
```

Otherwise, follow these rules:

- Pick a neutral sans-serif
- Ignore typefaces with fewer than 5 weights. They are usually not as well-crafted and are less
  popular. On Google Fonts, filter your search with **Number of styles** and select **10+**.
- Sort by popularity if you are unsure

Consider letter spacing and x-height. The *x-height* is the height of the lowercase letter x in a
given typeface:

- Smaller text should have higher x-height and wider letter spacing
- Fonts for larger text like headings have a shorter x-height and tighter letter spacing

### Line length

Paragraphs should be 45–75 characters wide. Apply the `ch` unit for exact characters, or apply
20–35em.

Real-world anchors: Medium's article body is constrained to roughly 680px, which hits the
65-character sweet spot at 18px. The Guardian constrains its article body to roughly 600px.

### Alignment

Always align by baseline, not by center.

### Line height

- Line height should be proportional to your line length. Shorter lines can apply 1.5, but longer
  lines may need close to 2.
- Small text needs more line height
- Larger text can apply much less, closer to 1

### Links

If a link is not on the main user path, you may not need special styles on it. An underline or a
color change on hover may be sufficient.

In app interfaces (not content sites), suppress underlines on navigation links and reserve the
underline for inline hyperlinks within body copy. GitHub applies this pattern: navigation labels
are unstyled, but links within issue bodies are underlined, making the distinction between
navigation and reference immediately clear.

### Numbers

Right-align numbers in a table so users can compare them by decimal place.

### Letter spacing

If you selected a popular typeface, trust its designer and leave the letter spacing alone.

Here are scenarios where you might consider changing the letter spacing:

- **Headings with fonts optimized for small sizes:** Adjust letter spacing to something like
  `-0.05em`. This strategy works only for fonts designed for small sizes. You cannot apply it to
  heading-specific fonts like Oswald and increase the spacing instead.
- **All-caps text:** Lowercase letters have visual variety and are easily distinguished from each
  other. They have ascenders and descenders that rise above the x-height and fall below the
  baseline. Uppercase letters are not as distinguishable. Increase letter spacing when you set text
  in all caps to compensate. Start with `0.05em`. Most badge and tag components (like GitHub's
  label chips or Stripe's status pills) apply this technique at small sizes.

## Color

> You cannot rely on color alone to communicate state. If color indicates a state, find a way to
> indicate it with color and an icon. People with color blindness cannot understand states
> communicated by color only.
>
> If you are using colors to show a difference (for example, in a pie chart), try applying lighter
> and darker shades of the same color to indicate the difference. People with color blindness can
> distinguish contrast differences rather than hue differences.

Prefer HSL (hue, saturation, lightness) over hex or RGB when writing color values. HSL represents
colors using attributes that match human visual perception:

- **Hue:** Position on the color wheel, expressed in degrees. 0 and 360 are both red, 120 is
  green, and 240 is blue.
- **Saturation:** How colorful or vivid the color looks. 0% is grey and 100% is vibrant and
  intense. A hue with 0% saturation appears grey regardless of its degree.
- **Lightness:** How close a color is to black or white. 0% is pure black, 50% is the pure hue,
  and 100% is pure white.

Modern CSS also supports `oklch()`, which improves on HSL by providing perceptually uniform
lightness. In HSL, increasing the lightness of a yellow shifts it far brighter than increasing the
lightness of a blue by the same amount. OKLCH (ok, lightness, chroma, hue) corrects this. Figma
and Tailwind CSS v3+ expose OKLCH values. It is worth knowing as a next step beyond HSL.

> Do not let lightness wash out your colors. Increase saturation as lightness increases beyond 50%.

### The 60-30-10 rule

A useful starting framework borrowed from interior design: allocate roughly 60% of your visual
space to neutral colors (backgrounds, panels), 30% to your primary color (navigation, headings,
key UI elements), and 10% to accent colors (calls to action, alerts, badges). These are not hard
limits, but they prevent any single color from dominating the layout.

### Categories

You always need more colors than you think. Your color palette should include colors from three
categories: greys, primary colors, and accent colors.

#### Greys

Greys cover text, backgrounds, panels, form controls, and borders. Build 8–10 shades and avoid
true black. Start with a dark grey and work up to white.

Your greys do not have to be neutral grey. True grey has 0% saturation. In practice, many colors
in designs look grey but are just very desaturated hues. You can make them warmer or cooler by
changing their temperature:

- **Cooler:** Saturate with blue
- **Warmer:** Saturate with yellow or orange

To maintain a consistent temperature, increase saturation for lighter and darker shades.

#### Primary colors

Primary colors determine the overall personality of the site. Apply them to primary actions, active
navigation elements, and focus states. Darker shades work well for text, and very light shades work
well for backgrounds and alert banners. Build 5–10 shades.

#### Accent colors

Accent colors draw the user's attention or communicate state: red for a destructive action, yellow
for a warning, or green for a positive outcome (think callout banners). Build up to 10 different
accent colors with 5–10 shades each.

Real-world palette examples show how these categories appear in production:

| Product | Greys | Primary | Accent |
|---|---|---|---|
| Stripe | Cool-tinted slate | Indigo | Green (success), red (error) |
| Linear | Near-black backgrounds | Violet | Minimal accent usage |
| Duolingo | White backgrounds | Green | Yellow, red, blue (per subject) |

### Changing brightness

When you increase a color's lightness value, the color moves closer to black or white and loses
intensity. A better approach is to *rotate the hue* to change perceived brightness while preserving
vibrancy:

- **Brighter:** Rotate toward the nearest bright hue: 60° (yellow), 180° (cyan), or 300° (magenta)
- **Darker:** Rotate toward the nearest dark hue: 0° (red), 120° (green), or 240° (blue)

### Text contrast

Text under 18px needs a contrast ratio of at least 4.5:1. Larger text needs a ratio of at least
3:1. Light text on dark backgrounds is problematic because achieving sufficient contrast requires a
very dark background, which can draw attention away from the content.

The solution is to flip the contrast: apply a light background with dark text. This approach does
not interfere with other interactive elements on the page.

### Colored text on colored backgrounds

For colored text on a colored background, adjusting lightness and saturation alone is often not
enough to achieve sufficient contrast. This often requires approaching near-white.

Rotate the hue closer to a brighter color: cyan (180°), magenta (300°), or yellow (60°). For
example, Stripe's success banners apply a green background at roughly `hsl(145, 63%, 42%)` with
near-white text. The hue rotation is what makes the contrast work without washing out the green.

## Depth and elevation

Shadows communicate an element's position on the z-axis. An element close to the surface casts a
light, diffuse shadow. An element that floats high above the surface casts a darker, tighter shadow.

Build a shadow scale the same way you build a spacing scale: define a small number of named levels
rather than choosing arbitrary values each time. The following variables show a five-level scale:

```scss
$shadow-sm:  0 1px 2px hsl(0 0% 0% / 0.05);
$shadow-md:  0 4px 6px hsl(0 0% 0% / 0.07), 0 2px 4px hsl(0 0% 0% / 0.05);
$shadow-lg:  0 10px 15px hsl(0 0% 0% / 0.10), 0 4px 6px hsl(0 0% 0% / 0.05);
$shadow-xl:  0 20px 25px hsl(0 0% 0% / 0.10), 0 10px 10px hsl(0 0% 0% / 0.04);
$shadow-2xl: 0 25px 50px hsl(0 0% 0% / 0.25);
```

Material Design's elevation system is the canonical real-world example: cards rest at elevation 1,
dropdowns at elevation 8, and modals at elevation 24. Each level has a defined shadow and, in
Material Design 3, a corresponding tonal color shift.

Two additional rules improve shadow realism:

- **Direction:** Real light comes from above. Set your `y-offset` positive (shadow falls downward)
  and your `x-offset` to zero or near zero.
- **Color:** Shadows are rarely pure black. Tinting the shadow with a slightly warm or cool hue
  (matching your grey temperature) makes it feel more natural.

## Interaction states

Every interactive element needs five states: default, hover, focus, active, and disabled.
Designing all five before shipping prevents jarring gaps in the experience.

GitHub's primary button is a good reference:

- **Default:** Green background, white label
- **Hover:** Slightly darker green background
- **Focus:** Green background with an offset focus ring (for keyboard navigation)
- **Active:** Darker green, slight inset appearance
- **Disabled:** Reduced opacity, `cursor: not-allowed`

Some rules for each state:

- **Hover:** A 5–10% lightness change is sufficient for most backgrounds. Avoid changing the size
  or layout on hover.
- **Focus:** Never remove the focus ring. Style it with `outline-offset` to keep it visually clean.
  For example, `outline: 2px solid hsl(220, 90%, 56%); outline-offset: 2px` gives a clean,
  accessible focus indicator.
- **Active:** A slight darkening and inward shadow reinforces the physical metaphor of pressing.
- **Disabled:** Reduce opacity (typically 40–50%) and remove pointer events. Do not simply grey out
  the color, as this can look like an unintentional design choice rather than a deliberate state.

## Iconography

Icons communicate faster than words, but inconsistently applied icons introduce visual noise. A
few rules keep icon usage coherent.

**Stroke weight:** Do not mix icon sets with different stroke weights in the same interface. For
example, Heroicons outline (1.5px stroke) and Feather (1px stroke) look mismatched at the same
size. Pick one set and apply it throughout.

**Size anchors:** Icons have two standard sizes in most design systems:

- 16px: Inline with text, inside buttons, or in tight UI chrome
- 24px: Standalone, in navigation, or as feature illustrations

Avoid scaling outlined icons below 16px. At small sizes, the strokes blur together and legibility
collapses. Switch to filled (solid) variants at 16px and below. Heroicons publishes both outline
and solid variants for exactly this reason.

**Optical alignment:** Icons are often designed with internal padding. When placing an icon next to
text, align to the visual center of the icon, not the bounding box. For example, a 24px icon may
have 2px of internal padding on each side, making the visual icon only 20px wide. Aligning to the
bounding box produces text that appears misaligned.
