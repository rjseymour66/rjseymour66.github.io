---
title: "Debugging"
# linkTitle: ""
weight: 120
# description:
---

## Testing tools

- [axe DevTools](https://www.deque.com/axe-accessibility-testing-tools/): Open source tool by Deque Systems. Its available as a [Chrome extension](https://chromewebstore.google.com/detail/axe-devtools-web-accessib/lhdoppojpmngadmnindnejefpokejbdd?hl=en-US), and here is the [Github repo](https://github.com/dequelabs/axe-core).
- Lighthouse: Google assessment tool and [CI GitHub action](https://github.com/marketplace/actions/lighthouse-ci-action)
- [WAVE](webaim.org): [Chrome extension](https://chromewebstore.google.com/detail/wave-evaluation-tool/jbbplnpkjmmeebjpijfedlgcdilocofh)
- [ARC Toolkit](https://www.tpgi.com/): [Chrome extension](https://chromewebstore.google.com/detail/arc-toolkit/chdkkkccnlfncngelccgbgfmjebmkmce)
- IBM Equal Access Accessibility Checker: [Chrome extension](https://chromewebstore.google.com/detail/ibm-equal-access-accessib/lkcagbfjnkomcinoddgooolagloogehp)
- [pa11y](https://pa11y.org/): A [CI tool](https://github.com/pa11y/pa11y?tab=readme-ov-file#command-line-interface)

## Chrome dev tools

### Accessibility Tree, ARIA Attributes, Computed Properties, and Source Order

Browsers have built-in tools to view these sections. Here is how you see it in Chrome:
1. Open DevTools
2. Select an element
3. Select **Accessibility** in the right pane

### Rendering

The Rendering pane lets you emulate color deficiencies, reduced motion, forced-colors, and more:

1. Open DevTools
2. Select the vertical ellipses.
3. Select **More tools** > **Rendering**.
4. Pick whatever you want to emulate.

## CSS for debugging

If you can't access any of the tools above, here are some styles that can draw attention to accessibility issues:

```css
/* highlight images with no alt */
img:not(alt) {
    border: 4px solid red;
}

/* test that the lang attr is set */
html:not([lang]),
html[lang*=" "],
html[lang=""] {
    border: 10px solid red;
}

/* CSS class that removes all color */
.a11y-tests-grayscale {
    filter: grayscale(100%) !important;
}

/* Blur entire page */
:root {
    --a11y-blur: 2px;
}

.ally-tests-blur {
    filter: blur(var(--a11-blur)) !important;
}
```