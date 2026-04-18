---
title: "Navigation"
# linkTitle: ""
weight: 90
# description:
---

Always use the `<nav>` element for navigation. It tells the screen reader and search engine that the section has a role of `navigation` which is a landmark.

Types of navigation:

- Global navigation: Links leading to top-level pages of the website that is found on every page. Can be displayed in nav bars, drop-down menus, and flyout menus.
  Changes according to viewport size
- Local navigation: Named anchors within text that link to other pages within the same website
- Breadcrumb: Local navigation that consists of a series of links that display the hierarchy of the current page in relation to the site's structure
- Page TOC:
- Sitemap: Page containing hierarchical links to every single page on the site

## "Skip to content" link

For better accessibility, let users skip blocks of content that appear on each page and go directly to the content.

```html
<a href="#main" class="skip-link button">Skip to main</a>
...
<main id="main"></main>
```

This link might look strange, so you can hide it from view unless when it gets focus. Use these styles to make this work:

```css
.visually-hidden:not(:focus):not(:active)
```

## Table of Contents

- On larger screens, the TOC is on the right pane
- On smaller screens, its at the top of the page
- Never change the tab order on nav items for large or small screens

- include an `aria-label` to describe the purpose of the nav. If the value of this attr is redundant, use the `aria-labelledby` attr with an `id` attr:

```html
<!-- aria-labelledby -->
<nav aria-labelledby="tocTitle">
  <p id="tocTitle">On this page</p>
  <ul role="list">
    ...
  </ul>
</nav>
<!-- aria-label -->
<nav aria-label="Table of Contents">
  <p>On this page</p>
  <ul role="list">
    ...
  </ul>
</nav>
```

From the MDN docs:

When using `aria-label`, you also need to consider `aria-labelledby`:

- `aria-label` can be used in cases where text that could label the element is not visible. If there is visible text that labels an element, use `aria-labelledby` instead.
- The purpose of `aria-label` is the same as `aria-labelledby`. Both provide an accessible name for an element. If there is no visible name for the element you can reference, use `aria-label` to provide the user with a recognizable accessible name. If label text is available in the DOM and it's possible to reference it for an acceptable user experience, prefer to use `aria-labelledby`. Don't use both on the same element because `aria-labelledby` will take precedence over `aria-label` if both are applied.

## Page breadcrumbs

Secondary navigation to help users understand where they are on a website:

- offer insight into your site organization
- users can get around without browser naviagtion
- portions of the breadcrumb are often links
- add `aria-label="breadcrumbs"` to the `<nav>` element

JS:

```js
const url = new URL("https://web.dev/learn/html/navigation");
const sections = url.hostname + url.pathname.split("/");
// "web.dev,learn,html,navigation"
```

HTML - make sure you don't create a link for the current page, and use the `aria-current` label:

```html
<nav aria-label="breadcrumbs">
  <ol role="list">
    <li>
      <a href="/">Home</a>
    </li>
    <li>
      <a href="/learn">Learn</a>
    </li>
    <li>
      <a href="/learn/html">Learn HTML!</a>
    </li>
    <li aria-current="page">Navigation</li>
  </ol>
</nav>
```

CSS - screenreaders don't see these icons, which is a best practice:

```css
[aria-label^="breadcrumbs" i] li + li::before {
  content: "";
  display: block;
  width: 8px;
  height: 8px;
  border-top: 2px solid currentColor;
  border-right: 2px solid currentColor;
  rotate: 45deg;
  opacity: 0.8;
}
```

## Local navigation

This is what displays in the left sidebar, often with a filter and links to each section:

- on mobile, hide this nav with a hamburger menu
- you can add an arrow at the bottom of mobile screens to jump to the top to get back to the hamburger menu, or the 'on this page' section

Use CSS to indicate the current page, and add `aria-current="page"` to the link

```html
<li>
  <a aria-current="page" aria-selected="true" href="/learn/html/navigation">
    Navigation
  </a>
</li>
```

## Global navigation

Leads to top-level pages of the site and is the same on all pages.

- can be tabs that open nested lists to the subsections
- can include buttons and search widgets
- add `aria-current="page"` to the current section
- footers should be identical on all pages too