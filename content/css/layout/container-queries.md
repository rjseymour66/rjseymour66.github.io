+++
title = 'Container Queries'
date = '2025-08-06T10:12:10-04:00'
weight = 60
draft = false
+++

Container queries make individual UI modules responsive based on their container's size or style, rather than the viewport. A _container_ is an ancestor element that contains the element you are sizing. It typically defines a region of the page through its dimensions, background, or other visual boundary.

There are two kinds of container queries:
- **Container size queries**: Adjust element styles based on the width of their container.
- **Container style queries**: Adjust element styles based on a custom property value on the container.

Without container queries, a module that does not fit its context at certain sizes forces you to write multiple `@media` queries, one for each breakpoint where the layout breaks. This couples the module to specific viewport sizes, which violates a key principle of modular design: **design modules independently of where you use them**. Container queries solve this by letting each module respond to its own container instead.

## Container size queries

Setting up a container size query takes two steps:
1. Define the container.
2. Query the container with the `@container` rule.

This example defines a container named `layout` and sets it to query on its inline size (width). The shorthand `container` property combines `container-name` and `container-type`:

```css
/* 1. Define the container */
.l-page > * {
  container-name: layout;
  container-type: inline-size;

  /* equivalent shorthand: */
  container: layout / inline-size;
}
```

After you define the container, use `@container` to query its width and apply styles to any element inside it, much like a `@media` query:

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

Container types control what can be queried on the container. CSS uses _containment_ to isolate a subsection of the DOM, which has an important side effect:

- You cannot use a container query to alter the size of the container being queried. This prevents a feedback loop: if you target an `h1` in a container `<= 300px` and the font size expands the container to `301px`, the query would no longer apply.
- To prevent this, you must set a `container-type` value on the element. The available values are:
  - `normal`: default; the element is not a container.
  - `inline-size`: the most practical option. Lets you query a container only on its inline size (width), not height. The height is determined by the contents of the container. Normally, the element width fills the available space, so this is the width you need to query against. For flex items, you need to set `flex-basis` or `flex-grow` or the container width is `0`.
  - `size`: has full-size containment in both block and inline directions. The height of the container is not determined by the contents. You need to set `height` or `min-height` explicitly, or use grid or flexbox to define the height. For absolute- or fixed-positioned elements, you can establish the height with the `inset` property. If you do not use any of these, the height is `0` and the container will not render correctly.

## Container names

Assign a container name so you can choose which container to query. You can assign unique names, or you can reuse the same name for multiple containers.

- **Reuse a name** like `layout` across containers. A container query targets the closest ancestor element with that container name, so reusing names lets different modules query their nearest matching ancestor independently.
- **Assign multiple names** to a single container when different modules need to query the same element by different names, depending on their context.
- **Assign no name** with caution: the browser walks up the DOM and queries the first container it finds. You also cannot use the `container` shorthand without a name.

You can assign multiple names to one container in a single declaration:

```css
/* multiple names, one query */
container: layout sidebar / inline-size;
```

### Containers and modules

Make a container when a module has an element that contains other modules. Apply this approach consistently across your stylesheet. Mixing container queries and `@media` queries for the same modules creates unpredictable layout behavior.

The following example makes a flex item into a container. Because the flex item needs to fill available space, it also sets `flex-grow: 1`:

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

You cannot use block-directional units (`cqh` and `cqb`) to query `inline-size` containers.

The following example sizes images and fonts using container units. It applies no container queries because the styles do not depend on breakpoints. They scale continuously with the container size:

```css
@layer modules {
  ...
  /* max height is 30% of container inline size */
  .race-detail > img {
    inline-size: 100%;
    max-block-size: 30cqi;
    object-fit: cover;
  }

  /* font is 3.5% of container inline size, with min and max */
  .race-detail__body > h3 {
    font-size: clamp(1rem, 3.5cqi, 1.8rem);
    font-family: Helvetica, Arial, sans-serif;
  }
}
```

## Container style queries

Container style queries adjust element styles based on a container's CSS custom property values. Any element that inherits the custom property can respond to it through a style query:

```css
@container style(--color-theme: dark) {
  /* styles */
}
```

You do not have to explicitly define a container type to use style queries:

- **Useful for context-dependent modules**: Apply different styles to a module based on where it appears on the page, without changing its HTML.
- **Progressive enhancement**: Container style queries are not yet widely adopted. Treat them as an enhancement rather than a core layout dependency.

The following example changes a `.race-detail` module's layout when it appears inside a sidebar. The layout layer sets a `--sidebar` custom property on any `aside` inside `.l-page`. The modules layer then reads that property with a style query and applies sidebar-specific styles:

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

This approach keeps the module portable. You can apply its sidebar styles to any page, regardless of context. The alternative couples the styles to a specific page structure:

```css
.l-page > aside .race-detail {...}
```

That selector only works when `.race-detail` is nested inside `.l-page`, limiting where you can reuse it.

### Dark mode with style queries

This pattern implements a theme switcher that respects the OS color preference by default and lets users override it with a button. Apply it with these steps:

1. Use a `prefers-color-scheme` media query to set a `--color-theme` custom property.
2. Add buttons that set the light and dark theme data attributes.
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

The following JavaScript sets the appropriate `data-theme` attribute when a user clicks either button:

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
