---
title: "Relative Units"
linkTitle: "Relative Units"
weight: 7
description: >
  Notes about basic Relative Units.
---

## Relative values

- A CSS pixel =/= to a monitor pixel
- **Computed value**: Absolute value that the browser computes for values declared using relative units
- Whe an element has a value defined using a length (px, rem, em, etc), its computed value is inherited by its child elements.

When to use rems:
- font sizing. Always use relative units when setting a font size because when users alter the screen zoom with + or -, pixels do not resize correctly.

When to use ems:
- padding
- margins
- element sizing
- border-radius

pixels:
- borders

percentages:
- container widths (as necessary)
