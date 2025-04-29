---
title: "Linking content"
# linkTitle: ""
weight: 40
# description:
---

Links should have a concise and descriptive text label, and users should know what to expect when they click or select it. Essential criteria includes the following:
- **Must convey its role**: Use the `<a>` tag with an `href` attribute. It has an implicit `link` role, and a screen reader announces the role with the text. Links **should not** have click events or placeholder text--that is for a button!
- **Have an accessible name**: Link text should be meaningful, short, understandable, and unique, or sincere, substantial, succinct, and specific. Don't use 'learn more' or 'click here'.
- **Unique label. Concise and straightforward**: Don't have multiple links with the same text on a single page.
- **Accessible to assistive tech**: it should communicate its current state (visited, focus-visible, hover, active)
- **Focusable with a keyboard**: By default, the `<a>` element is interactive and tabbable.

## Styles

Links should look like links--select a link color and underline them. Best practices for link styles:
- Underline links--don't rely on color alone to convey meaning. Older users and colorblind users cannot distinguish colors.
- Provide different styles for different states, including `focus` or `focus-visible`. `focus-visible` is specific to keyboard tabbing, while `focus` activates with a mouse and JS `.focus()`
  - `text-decoration` for underlines
  - `outline` for focus.
  - `box-shadow` is good for focus but does not work with forced-colors mode. If you use `box-shadow`, use a `transparent` outline because it displays in forced-colors mode
- Make sure the link target size is large enough for users to tap on small devices. Always focus on usability, not aesthetics.
  Touch targets like links should be 24px x 24px
- 

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

If a link starts a file download, tell users the format and file size before they download it:
- How to access the file
- What info in the file
- File format in case the user can't open the file on their device
- File size for users with slow connections

How to create a download link:
- Descripting name that has format and file size
- Add the `download` attribute. You can give this a value to name the downloaded file
- Optionally add download icon

```html
<a href="#" download>View our menu (PDF, 1.2MB)</a>
<a href="#" download="our-menu.pdf">View our menu (Adobe Illustrator)</a>
```

Add an icon with the `[download]` attribute:

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

These links open in the user's default email application. Create an email link by starting the `href` attribute with `mailto:`. You can include one or more email addresses. You can also populate the subject and the body with URL-encoded text.

Use the actual email address for the link, not generic text. This way, users can just copy the link text if they don't want to use their default email app or go through a context switch:

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
- Informative: Provides info that is relative to the main content of a page or section. The `alt` tag should give a brief description that provides the info you want to convey and its context.
- Decorative: Visual design only. These don't require `alt` text.
- Functional: Acts like a link or button and the `alt` tag serves as the text for the link or button.

Here are some examples. Remember a few things:
- If you do not provide a value for the `alt` tag, then the image will not be in the accessibility tree.
- You still need to provide a text alternative with an `aria-label`.
- If you don't provide an `alt` tag to hide the image, you should also include the `aria-hidden` label.

```html
<!-- home page img SVG -->
<header>
    <a href="/path/to/page">
        <img src="/path/to/svg" alt="Home Page" />
    </a>
</header>

<!-- SVG directly -->
<a href="/path/to/page">
    <svg aria-labelledby="submit" role="img">
        <title id="submit">Submit</title>
        ...
    </svg>
</a>

<!-- SVG not in accessibility tree option 1 -->
<a href="/path/to/page" aria-label="Accessible name">
    <img src="#" alt=""/>
</a>

<!-- SVG not in accessibility tree option 2 -->
<a href="/path/to/page" aria-label="LinkedIn">
    <svg aria-hidden="true">
        ...
    </svg>
</a>
```

## Context-change information

If you use `target="_blank"` to open a link in a new tab, you need to indicate that the link opens in a new tab so you don't annoy or confuse users.

In general, you should avoid opening new tabs so the user has control over their browser. However, there are exceptions:
- If the linked content contains information that should be read alongside the main content, like instructions for a form or other reference docs.
- The link opens a widget like a date picker.

As a best practice, add an `aria-label` to the link with text explaining that it opens in new tab:

```html
<a href="#" target="_blank">My website (opens in new tab).</a>
```

You might be tempted to use a the `::after` pseudo-element to add the content, but that is not picked up by translations and is dependent on CSS loading correctly.

## Add links to groups of elements (cards)

Cards can contain an image, a few lines of content, and a link to the content it describes. Which portion of the card should you make into a link?

The best solution available seems to be described on [Heydon Pickering's blog](https://inclusive-components.design/cards/). Basically, you add relative positioning to a card, then create an absolutely-positioned pseudo-element that stretches out over the entire card so its clickable like a button.

The main drawback of this is that you cannot highlight the text within the card:

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

For hover and focus styles, you need to use `focus-within` because it selects the parent of any element that has focus. This example uses progressive enhancement because if the browser supports `focus-within`, it gets the box shadow and doesn't need the standard `:focus` styles.

Don't combine the `hover` and `focus-within` styles in the event that `focus-within` is not supported:

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