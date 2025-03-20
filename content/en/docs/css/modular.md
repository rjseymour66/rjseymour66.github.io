---
title: "Modular CSS"
# linkTitle: ""
weight: 120
# description:
---

Modular CSS is when you encapsulate your CSS, which means you break the CSS into component parts--modules--where each part has its own styles and other modules can't interfere with the styles.
- Define a module for each component on your page: navigation, dialog, progess bars, etc

Has a single selector (class name, etc) so you can reuse it across the website. If there was a more specific selector (`#sidebar .classname`), then you can't reuse that as easily because its so specific to a portion of the page.


## Variations

Sometimes you need a class that builds on a module. For example, you can have a `.message` class and then a `.message--error` class that is a variation on the original class. This is a _modifier_. In this section, the block is the `message` class, and the modifiers are the `--<modifer>` portion. To use the modifier, add both the block class and the modifier class to the element:

```html
<button class="button button--large">Read More</button>
<button class="button button--success">Save</button>
<button class="button button--danger button--small">Cancel</button>
```

```css
@layer modules {
  .button {
    padding: 0.5em 0.8em;
    border: 1px solid #265559;
    border-radius: 0.2em;
    background-color: transparent;
    font-size: 1rem;
  }

  .button--success {
    border-color: #cfe8c9;
    color: #fff;
    background-color: #2f5926;
  }

  .button--danger {
    border-color: #e8c9c9;
    color: #fff;
    background-color: #a92323;
  }

  .button--small {
    font-size: 0.8rem;
  }

  .button--large {
    font-size: 1.2rem;
  }
}
```

For media modules, you don't have to assign an element class to everything. In the following example, the `h4` doesn't have a class name because it is already clear which element it selects. Don't use this on generic elements like `<span>`:

```css
@layer modules {
  .media {
    display: flex;
    gap: 1.5em;
    padding: 1.5rem;
    background-color: #eee;
    border-radius: 5px;
  }

  .media--right {
    flex-direction: row-reverse;
  }

  .media__image {
    align-self: start;
  }

  .media__body {
    overflow: auto;
    margin-block-start: 0;
  }

  .media__body > h4 {
    margin-block-start: 0;
    color: #333;
  }
}
```

## BEM

[Block Element Modifier](en.bem.info/methodology)

Block-Element-Modifier is a CSS methodology:
- Block: Main element of a module that has a descriptive, unique class name such as `message`. Think of it as a namespace for the elements and their modifiers.
- Element: Child element of the module. Described in the form `media__image`. Describes the element's purpose, such as a menu item.
- Modifier: Class name added to the block when creating a variant, such as `message--error`. Describes the appearance, state, or behavior of a block or element.

## Module composition

Each module should style exactly one thing. Ask yourself, "what is this module's responsibility?" If you answer with more than one responsibility, then it is multiple modules.

Preprocessors let you merge multiple files into a single CSS file. This means that the browser only makes one request for the styles. If you use a preprocessor, you should create a file for each CSS module and load them into the main page like this:

```css
@use 'reset'
@use 'global'
@use 'message'
@use 'button'
...
```

### Naming modules

Ask yourself what the module represents conceptually. Don't use `button--red` and `button--blue` to describe the modules in case the colors change in the future. Use what they represent instead: `button--danger` and `button--success`.


When naming items with BEM, it might help to first identify the following:
- container element
- descendent elements of the container. Name these using the double underscore.
  
  For example, if you have a button that is nested in a navbar that uses the class `nav`, you can identify the button as `nav__button` to ensure that the styles apply only to that button.

## CSS scope

Lets you restrict styles so they apply only to a specified portion of the page. Scope is enforced by the browser and controlled by the cascade.

In this example, you use the `@scope` keyword to create a scope for an element with the `.media` class. All rules nested under this `@scope` declaration apply to elements that are children of the `.media` element:

```css
@layer modules {
  @scope (.media) {             /* assigns '.media' to the 'scope' pseudoclass selector */
    :scope {
      display: flex;
      gap: 1.5em;
      padding: 1.5rem;
      background-color: #eee;
      border-radius: 5px;
    }

    .scope.right {
      flex-direction: row-reverse;
    }

    img {
      align-self: start;
    }

    .body {
      overflow: auto;
      margin-block-start: 0;
    }

    h4 {
      margin-block-start: 0;
      color: #333;
    }
  }
}
```

### Scope proximity

When you nest scopes, there might be situations where both scopes target the same element. First, the browser looks at the specificity, but if these values are equal, it uses proximity. Essentially, this means that the browser applies styles for the closest scope to the element. Its a way to ensure that a module is not adversely affected by a containing module.

### Scoping limit

Instead of relying on proximity, you can use a scoping limit to resolve conflicting styles. The following scope goes from the `.highlight-block` element to the `.slot` element. This resolves conflicting declarations between two scopes:

```css
@layer modules {
  @scope (.highlight-block) to (.slot) {
    :scope {
      padding: 1rem 1.5rem;
      background-color: #b3cbe6;
    }
    h1,
    h2,
    h3,
    h4 {
      color: #264b73;
    }
  }
}
```

The only naming collision issues come at the scope's name. You can distinguish scope classes by prepending them with an `m-` (for module), or you can use data attributes. To target a data attribute with a scope rule:

```css
@scope ([data-scope="dropdown"]) to ([data-scope])
```

## Pattern libraries

https://storybook.js.org/ is good if you are using one of its supported JS frameworks.

When you start writing CSS with modules, you'll find that you reuse much of the CSS without having to write new styles. This means that you have an inventory of CSS styles that you can document as a pattern library.


### CSS-first workflow

You should write your CSS before your HTML, and refer to your pattern library for modules:

1. Create a sketch or mockup of what the page should look like
2. Go to your pattern library and look for modules that can help. Start from the layout (outside) and work your way in.
3. If you need a module that doesn't exist, build it and add it to the pattern library.
4. Add modules to your stylesheet.

### Semver

Short for 'semantic versioning' (https://semver.org/). When you create a pattern library, you should version it when you make changes:

- Small changes, bug fixes: increment the patch number
- New module or functionality that doesn't break existing library: increment minor version number and reset patch to 0
- Substantial changes: increment the major version

This helps to keep you from just writing more and more CSS all the time. You remove unused parts of the stylesheet and add new ones, all while tracking your changes.