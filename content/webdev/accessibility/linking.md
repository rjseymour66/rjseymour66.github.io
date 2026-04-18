---
title: "Linking content"
# linkTitle: ""
weight: 40
# description:
---

Links should have a concise and descriptive text label, and users should know what to expect when they click or select one. Essential criteria include the following:

- **Must convey its role**: Apply the `<a>` tag with an `href` attribute. It has an implicit `link` role, and a screen reader announces the role with the text. Links should not have click events or placeholder text. That is what a button is for.
- **Have an accessible name**: Link text should be meaningful, short, understandable, and unique. Do not write 'learn more' or 'click here'. A screen reader user who browses links in a list out of context needs "Download the Q3 report (PDF)", not "Click here".
- **Unique label, concise and straightforward**: Do not have multiple links with the same text on a single page.
- **Accessible to assistive tech**: The link should communicate its current state (visited, focus-visible, hover, active).
- **Focusable with a keyboard**: By default, the `<a>` element is interactive and tabbable.

## Styles

Links should look like links. Select a link color and underline them. Best practices for link styles:

- Underline links. Do not rely on color alone to convey meaning. Older users and users with color blindness cannot distinguish color differences.
- Provide different styles for different states, including `focus` or `focus-visible`. The `:focus-visible` pseudo-class is specific to keyboard tabbing, while `:focus` activates with a mouse and JavaScript `.focus()`.
  - Apply `text-decoration` for underlines.
  - Apply `outline` for focus.
  - `box-shadow` works well for focus but does not display in forced-colors mode. If you apply `box-shadow`, also add a `transparent` outline because it becomes visible in forced-colors mode.
- Make the link target large enough for users to tap on small devices. Touch targets should be at least 24px by 24px.

The following CSS covers all link states:

```css
a:link {
  color: blue;
  text-decoration: underline;
}

a:visited {
  color: purple;
}

a:hover {
  color: green;
  text-decoration: dotted;
}

a:active {
  color: red;
}

a:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

/* forced-colors mode: box-shadow with transparent outline */
a:focus-visible {
  outline: 2px solid transparent;
  box-shadow: 0 3px 0 0 currentColor;
}
```

## Download links

If a link starts a file download, tell users what they will get before they commit to downloading it. This is especially important for users on slow connections or mobile data. Include the following information:

- How to open the file
- What information the file contains
- The file format, in case the user cannot open it on their device
- The file size for users on slow connections

Create a download link as follows:

- Write a descriptive name that includes the format and file size.
- Add the `download` attribute. You can give it a value to name the downloaded file.
- Optionally add a download icon.

```html
<a href="#" download>View our menu (PDF, 1.2MB)</a>
<a href="#" download="our-menu.pdf">View our menu (Adobe Illustrator)</a>
```

Add an icon automatically to any element with the `[download]` attribute:

```css
[download]::after {
  content: "";
  background: url("path/to/download.svg");
  block-size: 1em;
  display: inline-block;
  inline-size: 1em;
}
```

## Email links

These links open in the user's default email application. Create an email link by starting the `href` attribute with `mailto:`. You can include one or more email addresses and pre-populate the subject and body with URL-encoded text.

Apply the actual email address as the visible link text, not generic text like "contact us". This way, users who do not want to trigger their default email app can copy the address directly from the page:

```html
<!-- single email addr -->
To contact us, email us at <a href="mailto:email@example.com">support@example.com</a>.

<!-- multiple email addrs -->
To contact us, email us at <a href="mailto: email@example.com, email2@example.com">support@example.com</a>.

<!-- email + subject + body -->
To contact us, email us at <a 
    href="mailto:email@example.com?subject=Support%20Request?body=Customer%20ID%20123456789">
    support@example.com </a>.
```

## Link images

There are three types of link images:

*Informative*
: Provides information that is relative to the main content of a page or section. The `alt` text should give a brief description that conveys the information and its context.

*Decorative*
: Visual design only. These do not require `alt` text.

*Functional*
: Acts like a link or button. The `alt` text serves as the accessible name for the link or button.

The following examples cover the most common patterns. Remember these rules:

- If you leave the `alt` tag empty, the image is excluded from the accessibility tree.
- If you exclude the image from the tree, you still need to provide a text alternative with `aria-label`.
- If you omit the `alt` tag entirely to hide the image, also add `aria-hidden="true"`.

```html
<!-- Informative: home page logo linking to home -->
<header>
    <a href="/path/to/page">
        <img src="/path/to/svg" alt="Home Page" />
    </a>
</header>

<!-- Functional: inline SVG as link content -->
<a href="/path/to/page">
    <svg aria-labelledby="submit" role="img">
        <title id="submit">Submit</title>
        ...
    </svg>
</a>

<!-- Decorative: image excluded from tree, accessible name on the link -->
<a href="/path/to/page" aria-label="Accessible name">
    <img src="#" alt=""/>
</a>

<!-- Decorative: SVG icon excluded from tree, accessible name on the link -->
<a href="/path/to/page" aria-label="LinkedIn">
    <svg aria-hidden="true">
        ...
    </svg>
</a>
```

## Context-change information

If you apply `target="_blank"` to open a link in a new tab, tell users in advance so you do not surprise or disorient them.

In general, avoid opening new tabs because it removes the user's control over their browser. The Back button no longer works for the new tab, and screen readers announce nothing to indicate a new tab opened. Exceptions include:

- The linked content contains information the user should read alongside the main content, such as instructions for a form or reference documentation.
- The link opens a widget such as a date picker.

As a best practice, add the explanation directly in the link text:

```html
<a href="#" target="_blank">My website (opens in new tab).</a>
```

You might be tempted to apply the `::after` pseudo-element to inject the "opens in new tab" text, but that approach has two problems: it is not picked up by translation tools, and it depends on CSS loading correctly.

## Add links to groups of elements (cards)

Cards can contain an image, a few lines of content, and a link to the full content. Deciding which part of the card to make into a link requires a tradeoff.

The best pattern available is described on [Heydon Pickering's blog](https://inclusive-components.design/cards/). Add `position: relative` to the card, then create an absolutely positioned `::after` pseudo-element on the primary link that stretches to cover the entire card. This makes the whole card clickable while keeping a single, well-labelled link in the accessibility tree.

The main limitation of this pattern is that users cannot select text within the card:

```html
<div class="card">
        <h3>Example card element</h3>
        <p class="author">by <a href="#">Author's Name</a></p>
        <p>
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Unde, tenetur
          voluptatibus quia sequi deleniti aut non ipsa adipisci ex nemo. Optio
          ipsum debitis maxime placeat?
        </p>
        <p>
          Read <a class="card-link" href="http://example.com">How to do stuff</a>!
        </p>
      </div>
```

```css
.card {
  position: relative;
  padding: 2em 3em;
  border-radius: 15px;
  border: 2px solid blue;
  max-width: 45ch;
}

.card-link::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  cursor: pointer;
}
```

For hover and focus styles, apply `:focus-within` because it selects the parent of any element that has focus. This example applies *progressive enhancement*: if the browser supports `:focus-within`, the card gets the box shadow style. Older browsers that do not support `:focus-within` fall back to the standard `:focus` underline on the link.

Do not combine the `hover` and `focus-within` selectors in a single rule. If `focus-within` is not supported, the combined rule would also lose the hover style:

```css
.card a:focus {
  text-decoration: underline;
}

.card:hover {
  box-shadow: 0 0 0 0.25rem /* <color> */;
}

.card:focus-within {
  box-shadow: 0 0 0 0.25rem;
}

.card:focus-within a:focus {
  text-decoration: none;
}
```
