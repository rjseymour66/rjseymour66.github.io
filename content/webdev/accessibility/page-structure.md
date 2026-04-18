---
title: "Page structure"
# linkTitle: ""
weight: 30
# description:
---

## Navigation landmarks

Navigation is a major landmark, and a page can have several: a main navigation, a breadcrumb trail, a local table of contents, and a footer navigation. Without labels, screen reader users cannot tell them apart. Distinguish navigations with `aria-label` so users can confidently identify and jump to the one they need.

Add the `aria-current="page"` attribute to identify which nav link corresponds to the current page.

### Main nav

The main navigation is nested in the `<header>` element and identifies the current page with `aria-current="page"`:

```html
<header>
    <nav aria-label="Main">
    <ul>
        <li><a href="">Home</a></li>
        <li><a href="" aria-current="page">Products</a></li>
        <li><a href="">Team</a></li>
        <li><a href="">Contact</a></li>
    </ul>
    </nav>
</header>
```

### Breadcrumb nav

Breadcrumbs typically appear near the top of the `<body>` and show the user's position in the site hierarchy. Mark the current page with `aria-current="page"` so screen readers announce "you are here":

```html
<nav aria-label="Breadcrumb">
    <ol>
    <li><a href="">Products</a></li>
    <li><a href="">Pets</a></li>
    <li><a href="" aria-current="page">Dog toys</a></li>
    </ol>
</nav>
```

### Local nav

Apply this style of navigation for an in-page table of contents on long articles or documentation pages:

```html
<nav aria-label="Contents">
    <ul>
        <li><a href="">Company</a></li>
        <li><a href="">Licensing</a></li>
        <li><a href="">References</a></li>
    </ul>
</nav>
```

## Form landmarks

Forms can include critical site elements such as search boxes, filters, and login forms. Add a `role` attribute to promote them to page landmarks and make them directly accessible to screen reader navigation shortcuts.

Screen reader support for `search` and `form` landmarks is inconsistent. To provide the best coverage, combine the element with the `role` attribute until all major screen readers support both:

### form with search role

```html
<form action="" role="search">
    <label for="site-search">Search</label>
    <input type="text" name="" id="site-search" />
</form>
```

### search form

```html
<form action="">
    <label for="site-search">Search</label>
    <input type="text" name="" id="site-search" />
</form>
```

### login form

```html
<form action="" role="form" aria-labelledby="heading_login">
    <h2 id="heading_login">Login</h2>
    <div>
        <label for="username">Username</label>
        <input type="text" name="" id="username" autocomplete="username" />
    </div>
    <div>
        <label for="password">Password</label>
        <input type="password" name="" id="password" autocomplete="password" />
    </div>
</form>
```

## Distinguish landmarks with labels

If you have multiple landmarks of the same type on a page, add labels so users can differentiate them. To label a landmark, apply either `aria-labelledby` or `aria-label`. Label values should be one or two words. The text that gives assistive technology a name for an element is called an *accessible name*.

### aria-labelledby

This attribute references another element on the page as the label. It is preferred over `aria-label` because it references visible content rather than introducing new text:

```html
<nav aria-labelledby="pagination_heading">
    <h2 id="pagination_heading">Pages</h2>
</nav>
```

### aria-label

Apply this attribute when there is no visible element to reference with `aria-labelledby`:

```html
<nav aria-label="Main">
    ...
</nav>
```

## Main content structure

Complex pages require structure so users without a visual overview can still navigate them. Headings, lists, sections, and ARIA landmarks each serve a different organizational role. Apply them together to give the page a clear hierarchy.

### Sections

The `<section>` element creates a generic region that groups content thematically.

#### Generic sections

Without a label, `<section>` behaves like a `<div>` structurally, but they serve different purposes. A `<div>` is for styling or scripting. A `<section>` always groups thematically related content. Unlabeled sections have a role of `generic`.

Apply `generic` sections only when no other landmark type applies.

#### Region sections

When a `<section>` or `<div>` receives an accessible name through `aria-label` or `aria-labelledby`, it gets a role of `region`. This also promotes the element to a landmark, making it reachable by screen reader navigation shortcuts:

```html
<section aria-label="Keyword search">
    ...
</section>
```

Reserve this for sections that contain important content users should navigate to directly. Do not promote every section to a landmark, or users will face an overwhelming list of landmarks with no meaningful hierarchy.

A region typically starts with a heading. A `region` also identifies a scrollable area of the page. To allow keyboard users to scroll the area, add `tabindex="0"` to the container:

```html
<div role="region" aria-label="Code Sample" tabindex="0">
    <article>
        ...
    </article>
</div>
```

### Asides

An `<aside>` contains content related to the nearby content, such as a pull quote, a related-links section, or advertising. The `<aside>` element has a role of `complementary`.

### Articles

An `<article>` is a self-contained composition that could be distributed or reused independently. Common examples include:

- Blog posts
- Comments
- News articles
- Interactive widgets
- Product listings
- Forum posts

Third-party software like RSS feed readers extracts article content and displays it in other contexts. Proper `<article>` markup makes that extraction reliable.

### Lists

Group related content in ordered or unordered lists for these accessibility benefits:

- Screen readers announce the total number of items in the list and the index of the current item.
- Screen readers announce that an item belongs to a list of *n* items.
- Screen reader shortcuts let users jump between lists and between individual items.
- Screen readers can display all lists on a page and let users jump to any of them.

## Outline documents with headings

Headings create a navigable outline of the page. Screen reader users frequently navigate by jumping from heading to heading, the same way a sighted user scans section titles before reading. Follow these rules:

- The screen reader announces the heading content and its level.
- Do not skip heading levels when nesting. Going from `<h2>` to `<h4>` creates a gap in the outline.
- Always order HTML content so screen readers can follow it logically. Interface elements a user should interact with before your content, such as a cookie banner or language selector, should come before the content in the markup.
- If you need to reorder elements visually, apply CSS rather than changing the HTML order.
