---
title: "Container queries"
linkTitle: "xContainer queries"
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

### Container types

Container types work because CSS uses _containment_ to isolate a subsection of the DOM from the rest of the DOM.

- This means that you cannot use a container query to alter the size of the container being queried. This prevents this scenario: You're targeting an `h1` in a container <= 300px, then the font increases the container to 301px and then the container query no longer applies
- To prevent this, you have set a value for the `container-type` for the element to one of these values:
  - `normal`: default, the element is not a container
  - `inline-size`: most practical usage, lets you query a container only on its inline size (width), not height. The height is determined by the contents of the container. Normally, the element width fills the available space, so this is the width you need to query against. For flex items, you need to set `flex-basis` or `flex-grow` or the container width is `0`.
  - `size`: has full-size containment in both block and inline directions. The height of the container is not determined by the contents. You need to set `height` or `min-height` explicitly, or use grid or flexbox to define the height. For absolute- or fix-positioned elements, you can establish the height with the `inset` property. If you don't use any of these, the height is `0` and you will have issues.

## Container names

Assign a container name so you can choose which container to query. You can assign unique names, or you can reuse the same name for multiple containers.

- You can reuse a name like `layout`. This usually applies to the container you want to query, because a container query queries against the closest ancestor element assigned a container name.
- Assign multiple names, depending on the use case
- (Bad idea) Assign no name, and the browser looks up the DOM and queries the first container it finds. Can't use `container` shorthand if you don't assign a name.

```css
/* multiple names, one query */
container: layout sidebar / inline-size;
```

### Containers and modules

Good practice is to make a container when a module has a containing element that might house other modules. This approach is almost always better than working with `@media` queries, but you have to be consistent.

This example makes a container out of a flex item, so it adds a `flex-grow` value

```html
<aside>
  <h3>Running tips</h3>
  <div>
    <img />
    <div class="media__body">
      <!-------- container -------->
      <h4>Change it up</h4>
      <p>
        Don't run the same every time you hit the road. Vary your pace, and vary
        the distance of your runs.
      </p>
    </div>
  </div>
</aside>
```

```css
@layer modules {
  .media {
    padding: 1.5rem;
    background-color: #eee;
    border-radius: 5px;
  }

  .media__image {
    margin-inline: auto;
  }

  .media__body {
    container: layout / inline-size;
    flex-grow: 1;
  }
  ...;
}
```

## Container units

Containers have their own units that are similar to viewport width and height:

- `cqw`: 1% of container width
- `cqh`: 1% of container height
- `cqi`: 1% of container inline size
- `cqb`: 1% of container block size
- `cqmin`: Smaller of `cqi` or `cqb`
- `cqmax`: Larger of `cqi` or `cqb`

You can't use block-directional units (`cqh` and `cqb`) to query `inline-size` containers.

This example uses container units to size images and fonts within a container. There are no container queries because you aren't applying styles based on size, you are just using the size of the container to apply styles:

```css
@layer modules {
  ...
  /* image is 30% container height */
  .race-detail > img {
    inline-size: 100%;
    max-block-size: 30cqi;
    object-fit: cover;
  }

  /* font is 3.5% of container width, w min and max */
  .race-detail__body > h3 {
    font-size: clamp(1rem, 3.5cqi, 1.8rem);
    font-family: Helvetica, Arial, sans-serif;
  }
}
```

## Container style queries

Container style queries respond to a container based on aspects of its style, such as custom property values that set a dark theme. All styles that inherit this custom property apply the styles in this container query:

```css
@container style(--color-theme: dark) {
  /* styles */
}
```

You do not have to explicitly define containers to use style queries:

- good if you want a module to appear differently, depending on the context
- not widely adopted, best if used as progressive enhancement

These rulesets change the behavior of the media object when it is in a sidebar. The layout module says, "For any `l-page` element with a child element with the `aside` class, set the `sidebar` custom property to `true`." Then the container query says, "For any module where the `sidebar` custom property is `true`, apply the styles that match these selectors":

```css
@layer layout {
  .l-page > aside {
    --sidebar: true;
  }
}

@layer modules {
  @container style(--sidebar: true) {
    .race-detail {
      margin-block: 1em;
      display: flex;
    }

    .race-detail > img {
      flex: 0 0 30cqi;
    }

    .race-detail__body {
      margin-block-start: 0;
    }
  }
}
```

These styles help you maintain modularity, because now you can apply styles to specific elements in context, regardless of what page they are on. For example, you could create these styles with this rule:

```css
.l-page > aside .race-detail {...}
```

But that couples the styles to pages where the element is nested in `.l-page`.

### Dark mode with style queries

You can use data attributes to apply light and dark theme styles:

1. Use a `prefers-color-scheme` media query to set a `--color-theme` custom property.
2. Add buttons that add the light and dark theme data attributes.
3. Use container style queries to adjust specific styles depending on which custom property is applied to the page.

```css
@layer base {
  :root {
    --color-theme: light;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --color-theme: dark;
    }
  }

  [data-theme="light"] {
    --color-theme: light;
  }

  [data-theme="dark"] {
    --color-theme: dark;
  }

  @container style(--color-theme: dark) {
    body {
      background-color: #555;
      color: white;
      color-scheme: dark;
    }
  }
}

@layer modules {
  /* ... */
  .race-detail {
    container: layout / inline-size;
    border: 1px solid #ccc;
  }

  @container style(--color-theme: dark) {
    .race-detail {
      background-color: #333;
    }
  }
  /* ...  */
  .media {
    padding: 1.5rem;
    background-color: #eee;
    border-radius: 5px;
  }

  @container style(--color-theme: dark) {
    .media {
      background-color: #5f5f5f;
    }
  }
  /* ... */
}
```

Here is the Javascript to set the appropriate `data-theme` attribute:

```js
const lightThemeButton = document.querySelector("#light-theme");
const darkThemeButton = document.querySelector("#dark-theme");

lightThemeButton.addEventListener("click", () => {
  document.documentElement.setAttribute("data-theme", "light");
});

darkThemeButton.addEventListener("click", () => {
  document.documentElement.setAttribute("data-theme", "dark");
});
```
