---
title: "Page structure"
# linkTitle: ""
weight: 30
# description:
---

## Navigation landmarks

Navigation is a major landmark, and there are many kinds of navigations. Distinguish between navigations with the `aria-label` attribute so screen reader users can confidently navigate your website.

Add the `aria-current="page"` attribute to identify the nav link for the current page.

### Main nav

The main nav is nested in the `<header>` element and identifies the current page with `aria-current="page"`:

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

Breadcrumbs are typically near the top of the `<body>` and identify the current page with `aria-current="page"`:

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

Use this style of navigation for an in-page table of contents:

```html
<nav aria-label="Contents">
    <li><a href="">Company</a></li>
    <li><a href="">Licensing</a></li>
    <li><a href="">References</a></li>
</nav>
```

## Form landmarks

Forms can include critical site elements, such as search boxes, filters, or login forms. Add a `role` to them to promote them to page landmarks and make them easily accessible to screen readers.

Screen reader support for search and form landmarks is inconsistent. To provide the best accessibility, combine the element with the `role` attribute until there is support across all screen readers.

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

If you have multiple landmarks of the same type--for example, navigation landmarks--you need to add labels to them so users can differentiate between them.

To label landmarks, use either `aria-labelledby` or `aria-label`. Label values should consist of one or two words. Text that gives assistive technology a name (label) for an element is called an _accessible name_.

### aria-labelledby

This label references another element on the page. This label is preferred because it references content on the page, rather than introducing new text entirely.


```html
<nav aria-labelledby="pagination_heading">
    <h2 id="pagination_heading">Pages</h2>
</nav>
```

### aria-label

Use this when there is no element to reference with `aria-labelledby`.

```html
<nav aria-label="Main">
    ...
</nav>
```

## Main content structure

Web pages can be complex. If a user cannot view the screen to understand its layout, you need to create landmarks from elements and use elements like headings and lists to give the page structure for navigation.

### \<section>

This element creates a generic region that groups content thematically. Without a label, the `<section>` element is the same as a `<div>`, but they do not have the same use cases. For example, divs are used for styling or scripting purposes, while sections always group content. Sections without a label have a role of `generic`.

Use `generic` landmarks only when no other landmark type applies.

When a `<section>` or `<div>` is given an accessible name with `aria-label` or `aria-labelledby`, it have a role of `region`. This also **promotes the element to a landmark**:

```html
<section aria-label="Keyword search">
    ...
</section>
```
Do not do this often. Only promote sections or divs that have important information that users should navigate to directly.

A region typically starts with a heading, and its contents should be listed in a summary of the page. A `region` also identifies a scrollable area of the page. To allow keyboard users access to a scrollable area, you have to add a `tabindex=0` to the containing element:

```html
<div role="region" aria-label="Code Sample" tabindex="0">
    <article>
        ...
    </article>
</div>
```

### Asides

An `<aside>` contains content that is related to the nearby content, or it could be separate like a quote or advertising. An `<aside>` element has a role of `complementary`.