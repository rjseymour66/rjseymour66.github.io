---
title: "Assets"
linkTitle: "Assets"
weight: 90
description: >
  How to work with and process assets.
---

You can get resources from the `assets/` directory with `resources.GetMatch` or `resources.Get`. The following example gets the CSS file and processes it with [Pipes](https://gohugo.io/hugo-pipes/transpile-sass-to-css/):

```html
{{ $css := resources.GetMatch "index.scss" | resources.ToCSS}}
<link rel="stylesheet" href="{{ $css.Permalink }}">
```