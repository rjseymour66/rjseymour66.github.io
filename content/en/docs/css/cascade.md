---
title: "Cascade"
linkTitle: "Cascade"
# weight: 1
# description: 
---

Much of how to accomplish stylings with CSS depends on the context, so it is best to understand the priniciples of the language:


declaration
: A line of CSS that includes a property and a value.

declaration block
: A group of declarations inside curly braces that is preceded by a selector

ruleset | rule
: Selector plus declaration block is a ruleset or rule. Rule is less common.

at-rule
: Rules that begin with the `@` symbol, like `@import` or `@media`.

```css
/* ruleset */

selector {
  /* declaration block */
  property: value; /* declaration */
  property: value;

} 

/* Example */

body {
  color: black;
  font-family: Helvetica;
}
```

## Cascade

Most of this is covered here: https://rjseymour66.github.io/docs/css/getting-started/#cascading-stylesheets

Cascade determines how conflicting rules are applied to HTML:
- CSS is simple but can get complicated as the style sheet grows, so you want to write CSS rules in a predictable way:
- CSS is added to elements depending on context: if class A exists, add B styles; if element X is a child of element Y, add Z styles.

Cascade looks at these criteria when applying rules:

1. Stylesheet origin
   - Browser applies user-agent styles, then User styles to override
   1. Author stylesheet - what the CSS dev adds to the page. Also called "User styles"
   2. User stylesheet - added via browser extension, usually added to overcome a11y issues
   3. User-agent - Browser's default styles
      1. headings h1 - h6
      2. p tag top and bottom margins
      3. ol and ul styles
      4. link colors
      5. default font sizes
2. Inline styles
   <a href="/" style="color: white;"></a>
   - Can override inline styles with author `!important` rule
3. Layer
4. Selector specificity
5. Scope
6. Source order

### Overall order of cascade precedence

1. Important user-agent
2. Important user
3. Important author
4. Normal author
5. Normal user
6. Normal user-agent