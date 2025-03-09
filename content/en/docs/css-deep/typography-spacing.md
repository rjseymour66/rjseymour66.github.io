---
title: "Typography and spacing"
linkTitle: "Typography"
weight: 150
# description:
---

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