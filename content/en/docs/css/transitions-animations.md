---
title: "Transitions and animations"
linkTitle: "Transitions and animations"
weight: 2
description: >
  Transitions and animations in CSS.
---

## Transitions

A transition lets you animate the change of styles from one state to another. You can define the following:
- which property changes
- duration and timing of the change

For example, the following styles add a transition to a nav button that slowly increases its size from its static state to its hover state. By adding the `transform` property to the static button, it transforms into the hover state (`scale(1.25)`) when you hover the cursor over it:

```css
.nav__link:link,
.nav__link:visited {
    text-decoration: none;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 48px;
    width: 48px;
    border-radius: 50%;    
    border: solid 1px transparent;
    transition: transform ease-in-out 250ms;
}

.nav__link:hover,
.nav__link:focus-visible {
    border-color: black;
    outline: none;
    transform: scale(1.25);
    background: inherit;
}
```