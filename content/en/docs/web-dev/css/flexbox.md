---
title: "Flexbox"
# linkTitle: "CSS in Depth"
weight: 70
# description:
---

[Flex visual cheatsheet](https://flexbox.malven.co/)

Flexbox is short for Flexible Box Layout, and can help you arrange elements in a row or column, vertically center items, and make elements equal height. Flexbox allows you to define 1-D layouts. It works from the content out. Use it for rows or columns of similar elements. You don't need to set the size, because that is determined by the content.

When you set `display: flex;` on an element, it becomes a _flex container_:
- All direct child elements become _flex items_
  - flex items align side by side, left to right, in a row. Use `flex-direction` to change this.
  - size is determined by the contents. 
- By default, flex items align side by side, left to right
- A flex container takes up 100% of its available width, while the height is determined natually by its contents. The `line-height` of the text inside each flex item is what determines the height of each item.
- Margins between flex items do not collapse like in the regular document flow.

A flex container has two axises:
- _Main axis_, which goes the same direction as the flex-direction setting. Ex: `row` (default), it goes left to right. `column`, top to bottom.
- _Cross axis_ runs perpendicular to main axis

In general, start a flexbox with the following methods:
- Apply `display: flex` to the flex container.
- Set a `gap` or `flex-direction`, as needed.
- Declare `flex` values for the flex items to control their size, as needed.

## Flex container

Apply `display: flex` to an element to turn it into a _flex container_, and all of its children become _flex items_. By default, flex items are the same height, and the height is determined by their content.

> If you use `display: inline-flex`, all elements become flex items, but they are inline elements instead of block.
>
> You will rarely use `inline-flex`.

## Flex items

When arranged in a row, flex items are placed horizontally along the _main axis_, and vertically along the _cross-axis_. Column layouts have a vertical main axis and horizontal cross axis.

### Size

The `flex` property controls the size of the flex items along the main axis:
- Setting just one value sets the `flex-grow` property, and flex-shrink is set to 1, and flex-basis is set to 0. The flex-basis setting means there is no default width and the flex item grows according to available space.
- `flex: 0;` means that the flex item will not grow beyond its default size
- `flex: auto;` 

By default, flex is set to `flex: 0 1 auto` (`flex: <grow> <shrink> <basis>`). It consists of three properties: `flex-grow`, `flex-shrink`, and `flex-basis`:
- `flex-basis`: Defines a starting size of an element before `flex-grow` or `flex-shrink` are applied. Set this like you would set a width, with px, ems, percentages.
  - When you set `flex-basis` to `auto`, then flexbox looks to see if there is a defined `width`. If so, it uses the `width`. Otherwise, it uses the contents to size the item.
  - By default, this is set to `auto`. When set to `auto`, the browser checks if there is a `width` declared on the element. If there is a `width`, the browser uses the `width` setting. If there is no `width`, the browser determines the size by the item's contents. If you have a `width` set, but `flex-basis` is set to anything other than `auto`, then the browser ignores the `width` setting.
- `flex-grow`: When the browser calculates the `flex-basis`, it does not necessarily fill the width of the flex container. Any remaining space is consumed according to the `flex-grow` setting on each flex item.
  - If `flex-grow` is set to 0, the item will not grow beyond its `flex-basis` value. If it is set to any number other than 0, then the items grow until the space is used.
  - The `flex-grow` value acts as a weight. For example, if you set item A to `1` and item B to `2`, then item B will take up twice as much of the remaining space as item A.
- `flex-shrink`: Indicates whether it should shrink to prevent overflow in the event that the flex items' initial size is larger than the flex container.
  - `flex-shrink: 0;` means that the item does not shrink.
  - Items with a higher value shrink more than items with a lower value.

The shorthand `flex` property is usually all you need. For example, the following styles creates two flex items, one twice as gig as the other, with a gap:

```css
.flex {
    display: flex;
    gap: var(--gap-size);
}

.column-main {
    flex: 2;
}

.column-sidebar {
    flex: 1;
}
```

When you need flex items to grow to fill their container, set `flex` to any value other than `0`.

## Flex direction

Add the `flex-direction` style to the flex container:
- When you change the flex direction to `column`, that means that `flex-basis`, `flex-grow`, and `flex-shrink` apply to the element height, not width. **The flex container's height is still determined by the contents of its flex items when you use  `flex-direction: column;`.**
- Swaps the direction of the main-axis and the cross-axis.
- For `flex-direction` is a row, the `flex` property applies to the width of the container. When `flex-direction` is column, the `flex` property applies to the height of the container.

### Flex basis and flex directions

If you set `flex-basis` to anything other than `auto`, flexbox ignores any height setting and only uses the contents. This is because `flex-grow` and `flex-shrink` begin calculating the sizing values at 0. If the flex item is empty, then it will not have any height.

This is tricky when you are using `flex-direction: column`;. When arranged in a row, flex items take up the width of their container because they are block elements. When arranged in a column, flex items default to the height of their container because they are block elements. If there is no content, that height is 0.

Depending on the `flex-direction`, `flex-basis` refers to the following:
- row: width 
- column: height

## Flex container properties

Apply the following properties to the flex container:

```css
.flex-container {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    /* flex-flow is shorthand for flex-direction and flex-wrap */
    flex-flow: column wrap;
    /* justify-content uses the main axis. Think about setting text. */
    justify-content: space-between;
    /* align-* props use the cross-axis */
    align-items: flex-start;
    /* align-content works only when flex-wrap is enabled. It 
    controls the spacing of the flex rows along the cross axis. */
    align-content: stretch;
    gap: 1rem;
    row-gap: 1rem;
    column-gap: 1rem;
}
```

- `flex-wrap: column/column-reverse` only works when something constrains the container.

`justify-content`, `align-items`, and `align-content` all accept the following values:
- `flex-start`, `start`, `left`
- `flex-end`, `end`, `right`
- `center`
- `space-between`
- `space-around`
- `space-evenly`

## Flex item properties

Apply the following properties to the flex container:

```css
.flex-item {
    order: 3;
    flex-grow: 1;
    flex-shrink: 2;
    flex-basis: auto;
    flex: 1;
    /* uses cross-axis, overrides the container align-items
    value. Ignored if item has margin: auto on cross-axis. */
    align-self: flex-start;
}
```

`align-self` accepts the following properties:
- `auto`
- `flex-start`, `start`, `self-start`
- `flex-end`, `end`, `self-end`
- `center`
- `stretch`
- `baseline`


## Use cases

[Typical use cases](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Typical_Use_Cases_of_Flexbox)

### Cards

You can use flexbox to make cards that are all the same size. For example, if you have a row of cards with a button at the bottom, you can push the button all the way to the bottom by setting `flex-grow` and `flex-shrink` to `1`, and `flex-basis` to `auto`. You can also use the `flex: 1;` shorthand:

```html
<div class="cards">
  <div class="card">
    <div class="content">
      <p>This card doesn't have much content.</p>
    </div>
    <footer>Card footer</footer>
  </div>
  <!-- more cards... -->
</div>
```

```css
.card {
  display: flex;
  flex-direction: column;
}

.card .content {
  /* flex: 1 1 auto; */
  flex: 1;
}
```

Because the `div.content` is the only item that can grow, it grows to fill up its container. When `flex-direction` is set to `column`, the items grow to fill the height of the container.

### Forms and search bars

```html
<form class="example">
  <div class="wrapper">
    <input type="text" id="text">
    <input type="submit" value="Send">
  </div>
</form>
```

```css
.wrapper {
  display: flex;
}

.wrapper input[type="text"] {
  flex: 1 1 auto;
}
```