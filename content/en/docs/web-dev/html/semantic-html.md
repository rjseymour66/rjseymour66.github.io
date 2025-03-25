---
title: "Semantic HTML"
# linkTitle: ""
weight: 40
# description:
---

Semantic means "relating to meaning". Semantic HTML means using HTML elements to structure your content based on each element's meaning, not its appearance. It is the difference between using a `<div>` to wrap the website's navigation, or using the `<nav>` element.

**Always ask yourself, "Which element best represents the function of this section of markup?".** Don't worry about how the element looks, that is a job for CSS.

This is helpful for the developer and for automated tools (e.g. screen readers) to understand the markup with the Accessibility Object Model (AOM).

## Accessibility Object Model (AOM)

When the browser parses an HTML document, it builds these object models:

- DOM
- CSSOM
- AOM, also called an accessibility tree. This is a semantic version of the DOM

[Explore the AOM](https://developer.chrome.com/docs/devtools/accessibility/reference#explore-tree)

Semantic HTML builds an AOM with meaningful landmarks rather than general text descriptions. For example, main portions of the the page such as `<nav>` and `<footer>` are defined in the AOM as landmarks. This provides structure to the page makes sure important content is keyboard-navigable for screen readers.

## `role` attribute

Here is a [list of the ARIA roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles).

Describes the role that the element has in the context of the document. The `role` attribute is valid on all elements, so it is called a "global" attribute. It is defined by the WAI-ARIA specification.

The role is important to screen readers and other assistive technologies and search engines. It helps the technology know how to interact with it.

- Buttons, links, ranges, etc have _implicit roles_. These are automatically added to the keyboard tab sequence because they expect user action.
- An _explicit role_ is any role that you assign to an element, regardless of what the tag name implies. For example, you can add `role=button` to any element, but it does not behave as a button, so it is not activated by pressing Enter or the space key.

Always prefer the implicit role. In other words, don't make a random div a button with `role=button`. Just use the `<button>` element.