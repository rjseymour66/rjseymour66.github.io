+++
title = 'Theme Switcher'
date = '2025-08-07T11:28:14-04:00'
weight = 40
draft = false
+++

You can add dark and light themes with custom properties. First, create the theme colors with class styles on the `:root` element:

```css
:root.dark {
  --border-btn: 1px solid rgb(220, 220, 220);
  --color-base-bg: rgb(18, 18, 18);
  --color-base-text: rgb(240, 240, 240);
  --color-btn-bg: rgb(36, 36, 36);
}

:root.light {
  --border-btn: 1px solid rgb(36, 36, 36);
  --color-base-bg: rgb(240, 240, 240);
  --color-base-text: rgb(18, 18, 18);
  --color-btn-bg: rgb(220, 220, 220);
}

/* apply custom props to elements below */
```

Then, use JS to grab the root element (`documentElement`), and toggle the theme:

```js
let setTheme = () => {
  const root = document.documentElement;
  const newTheme = root.className === "dark" ? "light" : "dark";
  root.className = newTheme;

  document.querySelector(".theme-name").textContent = newTheme;
};

document.querySelector(".theme-toggle").addEventListener("click", setTheme);
```