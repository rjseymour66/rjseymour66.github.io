---
title: "Container queries"
# linkTitle: ""
weight: 130
# description:
---

Container queries help make a UI module responsive. A _container_ is an ancestor element that contains the element that you are trying to size. It usually contains some sort of sizing or background color that defines the region of the page.

There are two kinds of container queries:
- container size queries: Modify the styles of elements based on the width of their container element
- container style queries: Modify styles of elements based on a custom property of the container

In some cases, a module does not fit nicely on the page at specific screen sizes, which might cause you to create multiple media queries to address any issues. This violates a key principle of modular design: **design your module without concern for the context that you might use it in**. You can replace the multiple media queries with container queries.

## Container size queries

1. Define the container
2. Query the container with the `@container` rule


This example gives the container a name and specifies that you will query it based on its inline size or width:

```css
/* 1. Define the container */
container-name: layout;
container-type: inline-size;

/* equivalent to this: */
container: layout / inline-size;
```
After you define the query, you use the `@container` syntax to query its width and size any element within the container accordingly, much like a `@media` query:

```css
@layer layout {
  .l-page {
    margin-inline: 1rem;
  }

  .l-page > * {                             /* define the container */
    container: layout / inline-size;
  }
  ...
}

@layer modules {
...

  @container (width >= 450px) {         /* create the container query */
    .media {
      display: flex;
      gap: 1.5rem;
    }

    .media__image {
      align-self: start;
      margin-inline: revert;
    }
  }
}
```
