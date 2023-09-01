---
title: "Architecture"
linkTitle: "Architecture"
weight: 2
description: >
  Architecture and CSS.
---

Applications often turn a piece of functionality into a component to promote reuse throughout the application. This reuse might result in naming conflicts---to resolve such conflicts, you need a naming system that prevents style collisions.

## OOCSS

[Object-oriented CSS](https://github.com/stubbornella/oocss/wiki)

## BEM

[Block Element Modifier](en.bem.info/methodology)

BEM breaks the user interface into composable blocks:

```
block-name__elem-name_mod-name_mod-val
```

Block
: Describes the block's purpose, such as a class name for a header. Think of it as a namespace for the elements and their modifiers.
  ```css
  .menu {...}
  ```

Element
: Describes the element's purpose, such as a menu item. The class name for an element is block name, double underscore, element name:
  ```css
  .menu__item {...}
  ```

Modifier
: Describes the appearance, state, or behavior of a block or element. The modifier is separated from what it describes by a single underscore:

  ```css
  .menu_hidden {...}
  .menu__item_open {...}
  ```

### Strategy

When naming items with BEM, it might help to first identify the following:
- container element
- descendent elements of the container. Name these using the double underscore.
  
  For example, if you have a button that is nested in a navbar that uses the class `nav`, you can identify the button as `nav__button` to ensure that the styles apply only to that button.