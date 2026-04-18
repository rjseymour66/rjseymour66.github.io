---
title: "Debugging"
# linkTitle: ""
weight: 120
# description:
---

Accessibility testing requires multiple tools because no single tool catches every issue. Automated scanners catch roughly 30–40% of WCAG violations. The rest require keyboard navigation, screen reader testing, and visual inspection. A good testing workflow combines at least one automated tool, keyboard-only navigation, and a screen reader.

## Testing tools

The following tools cover different parts of the accessibility audit process:

- [axe DevTools](https://www.deque.com/axe-accessibility-testing-tools/): Open-source automated scanner by Deque Systems. It reports WCAG violations with no false positives, making it a reliable first pass. Available as a [Chrome extension](https://chromewebstore.google.com/detail/axe-devtools-web-accessib/lhdoppojpmngadmnindnejefpokejbdd?hl=en-US) and a [JavaScript library](https://github.com/dequelabs/axe-core) you can integrate into your test suite.
- Lighthouse: Google's built-in audit tool. It scores accessibility alongside performance and SEO. Run it from Chrome DevTools or as a [CI GitHub Action](https://github.com/marketplace/actions/lighthouse-ci-action) to catch regressions in pull requests.
- [WAVE](https://webaim.org): Visual overlay tool that annotates the page with icons showing errors, alerts, and structural information. Available as a [Chrome extension](https://chromewebstore.google.com/detail/wave-evaluation-tool/jbbplnpkjmmeebjpijfedlgcdilocofh). Useful for understanding how a page is structured from an assistive technology perspective.
- [ARC Toolkit](https://www.tpgi.com/): Detailed inspector from TPGi. It provides granular checks for ARIA, colour contrast, and keyboard accessibility. Available as a [Chrome extension](https://chromewebstore.google.com/detail/arc-toolkit/chdkkkccnlfncngelccgbgfmjebmkmce).
- IBM Equal Access Accessibility Checker: IBM's rule-based scanner with its own rule set that complements axe. Available as a [Chrome extension](https://chromewebstore.google.com/detail/ibm-equal-access-accessib/lkcagbfjnkomcinoddgooolagloogehp).
- [pa11y](https://pa11y.org/): A headless command-line scanner built for continuous integration pipelines. Run it against any URL as part of a [CI workflow](https://github.com/pa11y/pa11y?tab=readme-ov-file#command-line-interface) to block regressions before they ship.

## Chrome dev tools

### Accessibility Tree, ARIA Attributes, Computed Properties, and Source Order

Browsers have built-in tools to inspect the accessibility tree, which is what a screen reader actually reads. This view shows each element's computed role, name, and state after the browser has applied all CSS and JavaScript. Follow these steps to open it in Chrome:

1. Open DevTools.
2. Select an element.
3. Select **Accessibility** in the right pane.

### Rendering

The Rendering pane emulates color deficiencies, reduced motion, forced-colors mode, and other user preferences without changing system settings. This lets you test how your page looks to users with different needs without requiring a separate device or profile. Follow these steps:

1. Open DevTools.
2. Select the vertical ellipses.
3. Select **More tools** > **Rendering**.
4. Pick whatever you want to emulate.

## CSS for debugging

If you cannot access any of the tools above, the following styles highlight accessibility issues directly in the browser. Add them temporarily to a development stylesheet and remove them before shipping.

```css
/* Highlight images with no alt attribute */
img:not([alt]) {
    border: 4px solid red;
}

/* Test that the lang attribute is set */
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

.a11y-tests-blur {
    filter: blur(var(--a11y-blur)) !important;
}
```

## Recommended testing workflow

Run these checks in order. Each step catches issues the previous step cannot:

1. **Run an automated scanner** (axe or Lighthouse) to catch obvious violations quickly.
2. **Navigate with only the keyboard.** Tab through every interactive element. Verify that focus is always visible, that you can reach and activate every control, and that focus never gets trapped unexpectedly.
3. **Check color contrast** with the WebAIM Contrast Checker or the browser's built-in contrast tool in DevTools.
4. **Emulate user preferences** in the Rendering pane: test dark mode, forced-colors mode, and reduced motion.
5. **Test with a screen reader.** NVDA with Firefox or VoiceOver with Safari are the most common combinations. Verify that every interactive element has a meaningful announced name and that dynamic content updates are announced.
