---
title: "Headings and sections"
# linkTitle: ""
weight: 50
# description:
---

Use the right element so you don't have to add notes to your markup. It also means you don't have to add extra classes or IDs to target elements with CSS or JS. Classes and IDs add no semantic value for screen readers or search engines. It also means that you don't have to use the `role` attribute as much.

## Semantic landmarks

Don't overdo landmarks in your page. It might create too much noise for screen readers and actually hurt accessibility.

MDN docs about [WAI-ARIA Roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles#landmark_roles).

### Main landmarks

| Element     | Description                                                                                 |
| ----------- | ------------------------------------------------------------------------------------------- |
| `<header>`  | Contains introductory content, such as a site-wide banner or section header.                |
| `<nav>`     | Defines navigation links, like menus or table of contents.                                  |
| `<main>`    | Encloses the primary content of the page, ensuring screen readers skip repeated navigation. |
| `<section>` | Groups related content under a common theme, usually with a heading.                        |
| `<article>` | Represents a self-contained content piece, such as a blog post or news article.             |
| `<aside>`   | Contains content related to the main content, such as sidebars or callout boxes.            |
| `<footer>`  | Defines a footer section, often containing copyright, legal, or contact information.        |

### Complementary landmarks

| Element     | Description                                                                   |
| ----------- | ----------------------------------------------------------------------------- |
| `<form>`    | Wraps interactive elements for user input and data submission.                |
| `<figure>`  | Groups media elements, such as images or diagrams, often with a <figcaption>. |
| `<dialog>`  | Represents a modal or popup for user interaction.                             |
| `<details>` | Provides expandable/collapsible content for additional information.           |
| `<summary>` | Works with `<details>` to provide a heading for collapsible content.          |

## `<header>`

```html
<header>
  <h1>Machine Learning Workshop</h1>
  <nav>
    <a href="#reg">Register</a>
    <a href="#about">About</a>
    <a href="#teachers">Instructors</a>
    <a href="#feedback">Testimonials</a>
  </nav>
</header>
```

- `<header>` is a landmark with the implicit role of `banner` when it is a top-level element. When it is nested in a `<main>`, `<article>`, or `<section>` element, it is not a landmark--it only identifies the header for that section.

- `<nav>` identifies content as navigation. When it is nested in the site heading (`<header>`), it is the main navigation for the site. If it is nested in another element, such as `<section>`, it is the navigation for that section only.

## `<footer>`

The footer contains contact information. Use the `<address>` element to enclose contact info for persons or organizations, not the actual mailing address:

```html
<footer>
  <p>&copy;2022 Machine Learning Workshop, LLC. All rights reserved.</p>
  <address>
    Instructors: <a href="/hal.html">Hal</a> and <a href="/eve.html">Eve</a>
  </address>
</footer>
```

The footer is a landmark when it is the site footer that appears on every page. It has the implicit role of `contentInfo`. Like the header, it is not a landmark when it is nested in another element. When you nest a footer, Chrome's AOM displays it as `FooterAsNonLandmark`.

## Document structure

The "holy grail" layout is a header, two sidebars, and a footer. You might nest multiple `<article>` or `<section>` elements in the `<main>` element:

```html
<body>
  <header>
    <h1>Site Heading</h1>
  </header>
  <nav>Nav</nav>
  <main>
    <header>
      <h2>Page heading</h2>
    </header>
    <section>
      <h2>Subsection One</h2>
      <article>First post</article>
      <article>Second post</article>
    </section>
    <section>
      <h2>Subsection Two</h2>
      <article>Third post</article>
      <article>Fourth post</article>
    </section>
  </main>
  <aside>Aside</aside>
  <footer>Footer</footer>
</body>
```

### `<main>`

Identifies the main content of the document. There should only be one `<main>` per page.

### `<aside>`

Content that is indirectly related to the main content, usually in a sidebar or a callout. The `<aside>` has a implicit role of `complementary`.

### `<article>`

A complete, self-contained section of content that is independantly reusable.

### `<section>`

Generic, standalone section of a document where there is not a more specific semantic element to apply. Each `<section>` should have a heading.

A `<section>` is not a landmark unless it has an accessible name. Then, it is has the implicit role of `region`. Add an accessible name with `aria-lablledby` or `aria-label`:

```html
<!-- Internal headings best practice -->
<section aria-labelledby="setup-heading">
  <h2 id="setup-heading">Setup Instructions</h2>
  <p>Follow these steps to set up your project...</p>
</section>

<!-- No on-screen headings -->
<section aria-label="User Guide">
  <p>This guide covers everything you need to know...</p>
</section>
```

### `<h1>-<h6>`

https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements

Screen readers use heading to understand a page's content, but they do not outline the page structure.

Headings are ranked in order of importance, from `<h1>` to `<h6>`:

- When nested in a `<header>`, it is the heading for the application or site
- When nested in `<main>`, it is the heading for the page. It doesn't have to be nested in a `<header>` element within the `<main>` element.
- When nested in `<article>` or `<section>`, it is the heading for that subsection of the page.

Use `<h1>` as the main heading, `<h2>` for subsection headings, and so on. Do not skip heading levels.

#### Heading font size

The agent stylesheet applies default sizing depending on where the element is nested. You need to make this consistent with styles:

```css
h1 {
  margin-block: 0.67em;
  font-size: 2em;
}

/* where() does not override other h1 rules because 
it has 0 specificity */
:where(h1) {
  margin-block: 0.67em;
  font-size: 2em;
}
```
