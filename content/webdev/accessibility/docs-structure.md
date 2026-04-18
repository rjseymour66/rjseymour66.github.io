---
title: "Document structure"
# linkTitle: ""
weight: 20
# description:
---

## Natural language with `lang`

The `<html>` element should include the `lang` attribute, which defines the natural language of the page content:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
  ...
```

The value of `lang` must be a valid [BCP 47 language tag](https://appmakers.substack.com/p/bcp-47-language-codes-list?utm_campaign=post&utm_medium=web) composed of one or more subtags:

- *Script subtag*: Defines the writing system. For example, `lang="sr-Latn"` specifies Serbian written in the Latin script rather than Cyrillic.
- *Region subtag*: A two-character country code in uppercase. For example, `lang="en-GB"` specifies British English. Screen readers typically ignore the region subtag, but translation services and spell checkers rely on it.

You can also add `lang` to any element that encloses text in a different language. This helps screen readers switch to the correct pronunciation engine for that passage. Apply it sparingly, because it can interrupt the document flow. This example specifies that a paragraph is written in German:

```html
<p lang="de">
    Ich habe nichts verschwendet und wäre auch, ohne es zu bekennen, getrost
    der Ewigkeit entgegengegangen, wenn nicht diejenige, die nach meinem.
</p>

<!-- Style elements by lang attribute -->
<style>
:lang(de) {
  font-style: italic
}
</style>
```

### Benefits

Setting `lang` correctly benefits users in the following ways:

- **Screen reader pronunciation**: The screen reader switches to the correct language voice. Without `lang`, a screen reader configured for English will mispronounce French or German words.
- **Page translation**: Google Translate detects the page language from `lang` before translating.
- **Quotation marks**: Different languages render quotes differently. French, for example, renders `«quoted content»` when `lang="fr"` is set.
- **Hyphenation**: The browser applies language-specific hyphenation rules when you enable CSS `hyphens`. English and German hyphenate at different syllable boundaries.
- **Font selection**: The browser may select a language-appropriate font for scripts such as Chinese or Arabic.
- **SEO**: Search engines apply the `lang` value to localization and regional ranking.
- **Form controls**: The browser formats locale-specific controls correctly. For example, it displays a comma as the decimal separator for French currency (`19,99` rather than `19.99`).

You can also target elements by language in CSS:

```css
:lang(es) {
  /* ruleset */
}
```

## Describe the document with \<title>

Describe the topic or purpose of the page with a unique `<title>` in the `<head>` element:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    ...
    <title>Structuring documents</title>
  </head>
```

The title can also add context so users immediately understand where they are, even when navigating from a search engine or a shared link:

```html
<title>Checkout (step 1 of 2) - Webstore Name</title>               <!-- Describe process -->
<title>errors - Log in - Webstore Name</title>                      <!-- Errors after login attempt -->
<title>4 results for term "hats" - Webstore Name</title>            <!-- Search results -->
<title>Page 2 - Product Search Results - Webstore Name</title>      <!-- Search results page -->
```

Screen readers announce the page title first when a page loads. Most screen readers also provide a keyboard shortcut to re-read the title at any time. This means a well-written title orients users who arrive mid-flow, such as a user who clicks Back during a multi-step checkout and lands on an unmarked step. Adding context matters for these reasons:

- A user may arrive from an external resource like a search engine, or the page may change unexpectedly after a user action.
- Screen readers orient users by announcing the title. Most screen readers have keyboard shortcuts to announce the page title.
- Bookmarked pages need descriptive titles so users can identify them later.
- Search engines display the title in results.

### Best practices

Follow these guidelines when writing page titles:

- **Unique**: Every page title should be distinct. Users who open multiple tabs need to distinguish them at a glance.
- **Concise**: Keep titles short. The title is the first thing a screen reader announces on a page load.
- **Descriptive**: Prioritize the user's task over marketing or SEO keywords. Describe the page's purpose clearly.
- **Front-loaded**: Place the most relevant information first. Write "Contact - My Website", not "My Website - Contact".
- **Contextual**: If the page is one step in a sequence, name it accordingly. For example, "Checkout (step 1 of 2) - Webstore Name".

To supplement the `<title>` element, add even more context for social media with [open graph](/docs/web-dev/html/metadata/#open-graph).

## Viewport width

Viewport settings should allow users to zoom. This `meta` tag instructs the browser to set the viewport width to the device's available width:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

- `width=device-width`: Sets the viewport width to the device's physical screen width. If you set a fixed width larger than the screen, content overflows. If you set it smaller, content clips or causes scroll issues.
- `initial-scale=1.0`: Sets the default zoom level to 100%. Not all browsers default to this value, so always set it explicitly.

The viewport is the size of the *initial containing block*, which is the root `<html>` element. Users need to zoom for many reasons:

- Text is too small
- Viewing image details
- Selecting words to copy and paste
- Cropping animated elements out of view to avoid distraction
- The site was not properly designed responsively
- A browser bug altered the zoom level
- Users expect a pinch or spread gesture to zoom

### Avoid these properties

The following settings restrict or disable zoom. Safari and Samsung Internet ignore them, and Chrome and Firefox let users override them, but you should still avoid them:

- `maximum-scale`: Defines the maximum scale level. The default is 10 in most browsers. Leave it unchanged.
- `user-scalable`: Controls whether zooming is allowed. Setting `no` or `0` disables zooming entirely.

## Optimal asset loading

The `<head>` element can contain many tags, links, and scripts that the browser parses line-by-line. The order of these elements directly affects loading performance. A slow or render-blocking resource delays everything that follows it, which means users wait longer before they see any content. Getting this order right is a small change with a measurable impact on perceived performance.

The following template shows the recommended order, as [explained by Harry Roberts](https://csswizardry.com/ct/):

```html
<!DOCTYPE html>
<html lang="en">                                    
  <head>
    <meta charset="UTF-8" />                                        <!-- character encoding -->
    <meta                                                           
      name="viewport" 
      content="width=device-width, initial-scale=1.0" />            <!-- viewport meta tag -->
    <meta
      http-equiv="Content-Security-Policy"
      content="upgrade-insecure-requests" />                        <!-- CSP headers -->
    <title>Structuring documents</title>                            <!-- title -->
    <link rel="preconnect" href="#" />                              <!-- initiate connection to resource (i.e. CDN) -->
    <script src="" async></script>                                  <!-- async JS -->
    <style>
      @import "file.css";
    </style>                                                        <!-- CSS that includes @import -->
    <script src=""></script>                                        <!-- synchronous JS -->
    <link rel="stylesheet" href="build/styles.css" />               <!-- synchronous CSS -->
    <link rel="preload" href="" />                                  <!-- declare fetch reqs before rendering to prevent blocking -->
    <script src="" defer></script>                                  <!-- Deferred JS -->
    <script type="module" src=""></script>
    <link rel="prefetch" href="" />                                 <!-- declare fetch request for page navigation -->
    <meta name="description" content="" />                          <!-- tags, icons, open graphs, etc -->
  </head>
```

### General rules

Follow these guidelines to keep `<head>` loading fast and predictable:

- Remove unnecessary tags, such as low-priority scripts or redirects. Place them in the `<body>` instead.
- Self-host your static assets instead of relying on third parties. [This article](https://csswizardry.com/2019/05/self-host-your-static-assets/) explains why third-party requests add uncontrolled latency.
- Always [validate](https://validator.w3.org/) your HTML.
- Always place page metadata first, such as character encoding and viewport information.
- Nothing that blocks rendering should come before `<title>`.
- CSS blocks JavaScript, so place synchronous JavaScript before CSS.
- Do not apply `@import` inside CSS files. It creates a second request chain and delays rendering.
- Place SEO and social meta tags last.

## Landmarks for semantic structure

> Follow the [ARIA rules](https://www.w3.org/TR/using-aria/#firstrule).

*Landmarks* are regions that define the organization and structure of a web page. Screen reader users rely on them to jump directly to the section they need, the same way sighted users scan visual layout. Without landmarks, a screen reader user must listen to the entire page from the top to find the main content.

Define landmarks with the appropriate HTML element (*implicit* role) or with `role` (*explicit*) when no appropriate element exists. Prefer elements with implicit roles. You do not need to specify a `role` on elements that already carry one.

### Major landmarks

Every element on the page should belong to one of these three common landmarks: `banner` (`<header>`), `main` (`<main>`), and `contentInfo` (`<footer>`).

#### banner landmark

There should be one `banner` landmark on each page.

The `<header>` element carries a role of `banner` and contains site-wide content such as a logo, skip links, global navigation, secondary navigation, search, and any other content visible on all pages. If the `<header>` is nested inside an `<article>` or `<aside>`, it is not a `banner` landmark.

The banner does not have to appear at the top of the page. It can be placed in a sidebar.

#### main landmark

There should be one `main` landmark per page. It should be a direct descendant of the `<html>` or `<body>` element. Wrapping it in a `<div>` is acceptable when layout requires it.

For single-page applications (SPAs), hide all inactive `main` landmarks.

#### contentInfo landmark

You can have multiple `<footer>` elements, but designate only one as the `contentInfo` landmark.

It contains site-wide content such as copyright information, secondary navigation, and other links. If the `<footer>` is nested inside an `<article>` or `<aside>`, it is not a `contentInfo` landmark.

### ARIA roles

The following table maps semantic HTML elements to their implicit ARIA roles:

| Element   | ARIA Role       | Conditions                                 |
| --------- | --------------- | ------------------------------------------ |
| `header`  | `banner`        | Only when a direct descendant of `<body>`. |
| `nav`     | `navigation`    |                                            |
| `main`    | `main`          |                                            |
| `section` | `region`        |                                            |
| `form`    | `form`          |                                            |
| `search`  | `search`        | Or form with `role="search"`               |
| `aside`   | `complementary` |                                            |
| `footer`  | `contentInfo`   | Only when a direct descendant of `<body>`. |

### Benefits

Semantic landmarks provide these advantages for screen reader users:

- **Orientation**: The screen reader announces the landmark type when the user enters it
- **Navigation**: Screen reader users can jump between landmarks with keyboard shortcuts
