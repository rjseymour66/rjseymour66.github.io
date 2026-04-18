---
title: "Document geometry and scrolling"
linkTitle: "Geometry and scrolling"
weight: 70
description:
---

Understanding how the browser renders elements on the screen requires you to
understand a coordinate-based view of the document.

## Document coordinates and viewport coordinates

The `x` coordinate increases from left to right and `y` increases from top to
bottom. Every coordinate is measured relative to one of two reference frames.

*Document coordinates* measure position relative to the top-left corner of the
full rendered HTML document, regardless of scroll position.

*Viewport coordinates* measure position relative to the top-left corner of the
visible browser window, excluding tabs, menus, and toolbars. Viewport
coordinates are sometimes called window coordinates. An `<iframe>` element
acts as its own viewport.

You convert between the two using *scroll offsets*. When the document is smaller
than the viewport and no scrolling has occurred, document and viewport
coordinates are equal. As the user scrolls, they diverge. For example, if an
element has a document `y` coordinate of 100px and you scroll down 25px, the
element's viewport `y` coordinate is 75px. Similarly, if an element has a
document `x` coordinate of 300px and you scroll right 100px, the element's
viewport `x` coordinate is 200px.

JavaScript tends to use viewport coordinates because document coordinates are
unreliable for locating elements precisely. Three factors make document
coordinates difficult to work with:

- CSS `overflow` lets elements contain more content than their visible area.
- Elements with their own scrollbars act as independent viewports.
- Mouse and pointer events report viewport coordinates.

CSS position values each use a different coordinate system:

- **`position: fixed`:** Positions the element using viewport coordinates for
  `top` and `left`.
- **`position: relative`:** Positions the element relative to where it would
  appear in normal document flow.
- **`position: absolute`:** Positions the element relative to the document, or
  relative to a positioned ancestor if one exists. For example, if a container
  element has `position: relative`, a child with `position: absolute` is
  positioned relative to that container, which creates its own coordinate
  context.

## Querying element geometry

When the browser renders an element, it occupies one or more rectangles on
the screen. Block elements are always a single rectangle. Inline elements can
span multiple lines, so the browser represents them as an array of rectangles.

Two methods expose this geometry:

- **`getBoundingClientRect()`:** Returns a single `DOMRect` object covering
  the entire element, including border and padding but not margin. The `left`
  and `right` properties give the horizontal bounds and `top` and `bottom`
  give the vertical bounds. The difference between `left` and `right` is the
  width; the difference between `top` and `bottom` is the height.
- **`getClientRects()`:** Returns an array of `DOMRect` objects, one per
  rendered rectangle. Most useful for inline elements that wrap across lines.

## Get element at a point

`document.elementFromPoint(x, y)` returns the topmost element at the given
viewport coordinates. The following example logs the element under the mouse
cursor on every hover event:

```js
let coords = document.elementFromPoint(555, 490);
console.log(coords);    // <button class="btn">Button</button>

// log everything that a mouse hovers over
window.addEventListener('mouseover', (e) => {
    console.log(document.elementFromPoint(e.clientX, e.clientY));
});
```

## Scrolling

### `scrollTo()`

`scrollTo(x, y)` scrolls the window so that the specified point is at the
top-left corner of the viewport. If the point is outside the scrollable area,
the browser scrolls as close as possible.

To scroll to the bottom of the page, subtract the viewport height from the
total document height:

```js
let docHeight = document.documentElement.offsetHeight;
let viewHeight = window.innerHeight;
window.scrollTo(0, docHeight - viewHeight);
```

### `scrollBy()`

`scrollBy(x, y)` scrolls by an amount relative to the current scroll position.
The following example advances the page 5px every 500ms:

```js
setInterval(() => scrollBy(0, 5), 500);
```

### Smooth scrolling

Pass an options object instead of coordinates to either `scrollTo()` or
`scrollBy()` to enable smooth scrolling:

```js
let docHeight = document.documentElement.offsetHeight;
let viewHeight = window.innerHeight;

// scrollTo
window.scrollTo({
    left: 0,
    top: docHeight - viewHeight,
    behavior: "smooth"
});

// scrollBy
window.scrollBy({
    left: 0,
    top: 200,
    behavior: "smooth"
});
```

### `scrollIntoView()`

`element.scrollIntoView()` scrolls the document until the element is visible.
By default, it aligns the top edge of the element with the top of the viewport.
Pass `false` to align the bottom edge instead. For more control, pass an options
object:

- **`block`:** Controls vertical alignment. Accepts `start`, `end`, `nearest`,
  or `center`.
- **`inline`:** Controls horizontal alignment. Accepts `start`, `end`,
  `nearest`, or `center`.

```js
b.scrollIntoView({
    behavior: "smooth",
    block: "center",
    inline: "nearest"
});
```

## Intersection Observer

Scroll event listeners fire hundreds of times per second. The *Intersection
Observer API* is more efficient: the browser calls your callback only when an
element crosses a visibility threshold, with no polling required.

Common use cases:

- Lazy-loading images below the fold
- Triggering CSS animations when elements enter the viewport
- Infinite scroll: loading more content when the user reaches the bottom
- Analytics: tracking which sections a user actually sees

```js
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            entry.target.classList.add('visible');
            observer.unobserve(entry.target);   // stop watching once visible
        }
    });
}, {
    threshold: 0.15,                    // fire when 15% of the element is in view
    rootMargin: '0px 0px -60px 0px',    // shrink the bottom of the viewport by 60px
});

document.querySelectorAll('.fade-in').forEach(el => observer.observe(el));
```

The `IntersectionObserver` constructor accepts an options object with the
following properties:

| Option | Description |
|--------|-------------|
| `threshold` | A number (0–1) or array of numbers. The fraction of the element that must be visible to trigger the callback. |
| `rootMargin` | Grows or shrinks the intersection root using the same syntax as CSS `margin`. Use negative values to trigger before an element fully enters the viewport. |
| `root` | The scroll container to observe relative to. Defaults to the browser viewport. |

### Real-world example: lazy-loading images

Swap a placeholder `src` for the real image URL only when the image scrolls
into view. This reduces initial page load time significantly:

```js
const imageObserver = new IntersectionObserver((entries) => {
    entries.forEach(({ isIntersecting, target }) => {
        if (!isIntersecting) return;

        target.src = target.dataset.src;
        target.removeAttribute('data-src');
        imageObserver.unobserve(target);
    });
});

document.querySelectorAll('img[data-src]').forEach(img => imageObserver.observe(img));
```

```html
<!-- Low-res placeholder loads immediately; real image loads on scroll -->
<img
  data-src="/images/product-hero.jpg"
  src="/images/placeholder-blur.jpg"
  alt="Product hero"
  width="1200" height="600"
>
```

### Real-world example: scroll progress bar

A reading progress bar shows how far down the page the user has scrolled. Use
`{ passive: true }` so the scroll listener never blocks rendering:

```js
const bar = document.querySelector('#progress-bar');

window.addEventListener('scroll', () => {
    const scrolled = window.scrollY;
    const total = document.documentElement.scrollHeight - window.innerHeight;
    bar.style.width = `${Math.round((scrolled / total) * 100)}%`;
}, { passive: true });
```

```html
<div id="progress-bar" style="
  position: fixed; top: 0; left: 0; height: 3px;
  background: #0057ff; width: 0%; transition: width 0.1s linear;
"></div>
```

## Viewport size, content size, scroll position

### Browser window size

Three categories of measurement describe the browser window and its content.

**Viewport size** is the visible area of the browser window, reported by
`window.innerWidth` and `window.innerHeight`. The HTML `<meta name="viewport">`
tag in the document head tells the browser the intended viewport width.

**Document size** is the full rendered size of the content, measured through
`document.documentElement`. Retrieve the width and height with either of the
following:

- `document.documentElement.getBoundingClientRect()`
- `document.documentElement.offsetWidth` and `document.documentElement.offsetHeight`

**Scroll offsets** report how far the document has been scrolled:
`window.scrollX` and `window.scrollY`. These properties are read-only. To
scroll the document programmatically, use `window.scrollTo()`.

The following example shows how to read all three measurements:

```js
// viewport size
wh = window.innerHeight;
ww = window.innerWidth;

// document size
docH = document.documentElement.getBoundingClientRect().height;
docW = document.documentElement.getBoundingClientRect().width;

dH = document.documentElement.offsetHeight;
dW = document.documentElement.offsetWidth;

// scroll offsets
wX = window.scrollX;
wY = window.scrollY;
```

### Element sizes

Element sizing is complicated. Every element exposes three groups of sizing
properties.

**Offset properties** measure the on-screen size and position of the element,
including border and padding:

- **`offsetWidth`:** On-screen width in pixels
- **`offsetHeight`:** On-screen height in pixels
- **`offsetLeft`:** `x` coordinate relative to the positioned parent element
- **`offsetTop`:** `y` coordinate relative to the positioned parent element
- **`offsetParent`:** The element that the offset properties are measured
  relative to

**Client properties** are all read-only and measure the content area and
padding, excluding border and scrollbar:

- **`clientWidth`:** On-screen width in pixels, content area and padding only
- **`clientHeight`:** On-screen height in pixels, content area and padding only
- **`clientLeft`:** Rarely useful. Measures the horizontal distance between
  the outer edge of the element's padding and its border.
- **`clientTop`:** Rarely useful. Measures the vertical distance between the
  outer edge of the element's padding and its border.

**Scroll properties** describe the element's scrollable content:

- **`scrollWidth`:** The element's content area, padding, and any overflowing
  content. Equal to `clientWidth` when there is no overflow.
- **`scrollHeight`:** The element's content area, padding, and any overflowing
  content. Equal to `clientHeight` when there is no overflow.
- **`scrollLeft`:** Writable. The horizontal scroll offset of the element's
  content within its viewport.
- **`scrollTop`:** Writable. The vertical scroll offset of the element's
  content within its viewport.
