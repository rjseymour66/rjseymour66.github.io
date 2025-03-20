---
title: "Document structure"
# linkTitle: "CSS in Depth"
weight: 20
# description:
---

And HTML document includes the `<html>` element, which also contains the `<head>` element and `<body>` element. The `<head>` element contains all the meta info about your page, like info for search engines, social media, icons for the browser tab, and behavior and presentation of your content with the character set and human language.

## Essential features

`<!DOCTYPE html>`: Tells the browser to use standards mode, as opposed to [quirks mode](https://developer.mozilla.org/en-US/docs/Web/HTML/Quirks_Mode_and_Standards_Mode), when rendering the document. Quirks mode is a leftover from before W3C web standards, where the browser had to render Netscape Navigator or Internet Explorer pages that did not follow a common HTML standard.

`<html lang="en">`: The root element for the HTML document that contains the `<head>` and `<body>` tags. The `lang` attribute accepts a two- or three-letter ISO code. Include the region if you can.

You can also use the `lang` attribute in other elements, if they need to be rendered in that language.

### `<head>` content

`<head>` contains all site or application metadata:

```html
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Document</title>
</head>
```

It can include the following:

- Character encoding: This needs to come before the `<title>` element so the browser knows how to render it. The default encoding is `windows-1252`, but you need to set it to `UTF-8`, which is required by HTML5.
- Title: You need a title--it is displayed in browser history, search results, social media cards, etc.
- Viewport metadata: Helps site responsiveness. `content="width=device-width` tells the browser to make the site responsive, and display the content using the width of the current screen. You can add the `initial-scale` and `user-scalable` attribute, but those use sensible, accessible defaults and are not necessary.

### Stylesheets

Add a CSS stylesheet `<link rel="stylesheet" href="style.css">`. This is the preferred way to include styles in your site. Keeping a separate stylesheet helps you maintain your site, and it allows the browser to cache it to enhance performance. `rel` tells the browser that the linked page has a stylesheet "relationship" to the HTML. You no longer have to put `type="text/css"`.

If you decide that you need to add styles directly in the HTML page, nest the `<style></style>` tags in the `<head>` as well.

### `<link>`

The `<link>` element is used to create a relationship between the HTML document and external resources. Here are the most common use cases:

- Favicons: `<link rel="icon" sizes="16x16 32x32 48x48" type="image/png" href="/images/mlwicon.png" />`

  `rel=icon` identifies the external file as the favicon for your document. The favicon always remains visible, regardless of the browser tab size.

  `sizes` tells the browser to use the linked favicon file when the provided dimensions are appropriate. if you have a scalable favicon, you can use `sizes=any`.

  Safari for iOS and macOS use special kinds of favicons. iOS has `rel="apple-touch-icon"`, which is used when a user pins the site to their homescreen. macOS uses `rel="mask-icon"` when the user pins the tab in Safari. This icon needs to be monochrome and requires the `color` attribute:

  ```html
  <link rel="mask-icon" href="/images/mlwicon.svg" color="#226DAA" />
  ```

- Alternate site versions: Use `rel="alternate"` to identify translations or alternate representations fo a site. This also requires that you set the `hreflang` attribute. For example:

  ```html
  <link
    rel="alternate"
    href="https://www.machinelearningworkshop.com/fr/"
    hreflang="fr-FR"
  />
  <link
    rel="alternate"
    href="https://www.machinelearningworkshop.com/pt/"
    hreflang="pt-BR"
  />
  ```

- Canonical site: When you have multiple versions of a site, use `rel="canonical"` to identify which site is preferred for search engine indexing:

  ```html
  <link rel="canonical" href="https://www.machinelearning.com" />
  ```

### `<script>`

Place `<scripts>` at the end of the document so they don't block the HTML rendering or asset downloading. Or, use the `defer` keyword to download the JS behind the scenes and execute when the page renders. `async` pauses the page rendering while it executes.

### `<base>`

Lets you set a default link URL and target. There can be only one per document, and it must come before any other relative URLs (`<link rel="something">`) in the `<head>`. This is not used often.

```html
<base target="_top" href="https://machinelearningworkshop.com" />
```

`target` determines how links open. Usually, you add `target` to individual links, not the entire page , but you can set the `target` attribute to one of the following:

- `_self`: opens linked files in the same context as the current document - the link opens in the current tab
- `_blank`: opens every link in a new window or tab, depending on the browser setting
- `_parent`: equivalent to `_self` if its not an iframe. If it is in an iframe, it opens the window in the parent window or tab
- `_top`: opens in the same browser tab but popped out of any context (iframe) into the entire tab.
