---
title: "Images"
linkTitle: "Images"
weight: 5
description: >
  Getting started with images in CSS.
---


## Sizing

To change the size of an image without changing its proportions, set the `width` to the size you want, and set the `height` to `auto`:

```css
img {
  height: auto;
  width: val;
}
```

Setting the height and width on an image also helps the browser calculate the image size while it loads the other content. This might prevent strange rendering behavior during page loads.
## Properties

### object-position

Works with `object-fit`, which tells the browser to calculate the optimum size of the image based on the dimensions provided so that it does not distort. `object-position` changes where the image is positioned inside the container to manipulate which part of the image is clipped.