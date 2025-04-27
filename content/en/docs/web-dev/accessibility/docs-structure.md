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
- script subtag: defines the writing system used. Ex: `lang="sr-Latn"`
- region subtag: 2-character country code in caps. Ex: `lang="en-GB"`. These are typically ignored by screen readers.

You can also add this to HTML elements that enclose text in another language. This helps screen readers with pronunciation, but uses it sparingly because it can interrupt the document flow. This `lang` attribute specifies that the paragraph is in German:

```html
<p lang="de">
    Ich habe nichts verschwendet und wäre auch, ohne es zu bekennen, getrost
    der Ewigkeit entgegengegangen, wenn nicht diejenige, die nach meinem.
</p>

<!-- Styling elements with lang attr -->
<style>
:lang(de) {
  font-style: italic
}
</style>
```
### Benefits

- Helps with screen reader pronunciation 
- Page translation: Google uses the `lang` tag to detect page language
- Quotes: different languages use different quotes. Ex: French uses `<<quote content>>`
- Hyphenation styles: If you add hyphens to the document, the language might add hyphens differently by langauge
- Font selection: The browser might select a language-appropriate font for languages like Chinese
- SEO: helps search engines with localization
- Form controls: Helps the browser format form controls. For example, using a comma for French currency rather than a decimal (`19,99`)


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

The title can also add context:

```html
<title>Checkout (step 1 of 2) - Webstore Name</title>               <!-- Describe process -->
<title>errors - Log in - Webstore Name</title>                      <!-- Errors after login attempt -->
<title>4 results for term "hats" - Webstore Name</title>            <!-- Search results -->
<title>Page 2 - Product Search Results - Webstore Name</title>      <!-- Search results page -->
```

We want to add context because for the following reasons:
- User might come from an external resource like a search engine, or the page might change unexpectedly after a user action.
- They help screen readers orient the user. Most screen readers have keyboard shortcuts to announce the page title. 
- Good for bookmarked pages
- Search engines use it in results

To supplement the `<title>` element, you can add even more context for social media sites with [open graph](/docs/web-dev/html/metadata/#open-graph).

### Best practices

- Titles should be unique. Users should be able to ID each page if they have multiple tabs open. This is more [complicated with an SPA](https://hidde.blog/accessible-page-titles-in-a-single-page-app/).
- Make it concise. The title is the first thing a screen reader encounters when reading a page.
- Be descriptive. SEO is important, but not as important as the user experience. Describe the page's purpose and omit any marketing or SEO terms that improve page ranking.
- Relevant information comes first. Use "Contact - My Website", not "My Website - Contact"
- Add context. If the page is one of a set of pages in a step, name it accordingly. For example, "Checkout (step 1 of 2) - Webstore Name".

## Viewport width

Viewport settings should allow users to zoom. This `meta` tag instructs the browser to use the available width of the device as the width for the viewport:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```
- `width=device-width`: Sets the viewport's width to the device's available width. Always set `width` to `device-width`. If you set the width larger than the screen, there will be overflow. If you set it smaller, you might clip content or have scroll issues.
- `initial-scale=1.0`: Sets the default zoom level to 100%. This might not be the default in browsers, so always set it explicitly.

The viewport is the size of the _initial containing block_, which is the root `<html>` element. Users need to zoom for many reasons:
- Text is too small
- View image details
- Select words to copy and paste easier
- Crop animated elements out of view to avoid distraction
- Site was not properly designed responsively
- Browser bug that impacts zoom level
- Users expect a pinch or spread gesture to zoom

### Avoid these properties

These settings negatively affect zoom. Safari and Samsung Internet of Android ignore settings, and Chrome and Firefox let you force zoom, but you should still avoid them:

- `maximum-scale`: Defines the maximum scale level. Default is 10 in most browsers. Leave it alone.
- `user-scalable`: Whether zooming is allowed. `no` or `0` disables zooming.- 

## Optimal asset loading

The head tag can contain many different tags, links, scripts, etc that it parses line-by-line. Their order can impact loading performance if a resource takes too long to load and blocks the browser from parsing subsequent tags. Here is the ideal order of the elements, as [explained by Harry Roberts](https://csswizardry.com/ct/):


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

- Remove unneccessary tags, such as low-priority scripts or redirects. You can place them in the `<body>`.
- Self host your static assets instead of relying on 3rd parties. [This article](https://csswizardry.com/2019/05/self-host-your-static-assets/) provides a detailed explanation.
- Always [validate](https://validator.w3.org/) your HTML.
- Always place page metadata first, such as character encodings and viewport information.
- Nothing that blocks rendering should come before `<title>`.
- CSS blocks JS, so place synchronous JS before CSS.
- Do not use `@import` in your CSS.
- SEO and social meta tags are last.

## Landmarks for semantic structure

> Follow the [ARIA rules](https://www.w3.org/TR/using-aria/#firstrule).

You need to include semantic regions so users can understand how the page is structured. Landmarks are regions that represent the organization and structure of a web page.

You can define landmarks with the appropriate HTML element (implicit) or use `role` (explicit) if no appropriate element exists. Prefer elements with implicit roles, rather than elements with explicit roles. You don't need to specify a role on on elements with implicit roles.

### Major landmarks

Landmarks are different from page regions. Every element in the page should be in one of these common landmarks: `banner` (`<header>`), `main` (`<main>`), and `contentInfo` (`<footer>`).


#### banner landmark

There should be one `banner` landmark on each page.  

The `<header>` has as a role of `banner` and contains site-oriented content such as a logo, skip links, main/global navigation, secondary nav, search, and any other content that is visibile on all pages. If the `<header>` is nested in an element such as `<article>` or `<aside>`, then it is not a `banner` landmark.

The banner doesn't need to be at the top--it can be in a sidebar.

#### main landmark

There should be one `main` landmark on the page, and it should be a direct descendant of the `<html>` or `<body>` element. It's OK to wrap it in a div, if necessary.

For SPAs, hide all inactive `main` landmarks.

#### contentInfo

You can have multiple `<footer>` elements, but you should designate only one as a `contentInfo` landmark.

Contains site-oriented content like copyright data, secondary nav, other links. If the `<footer>` is nested in an element such as `<article>` or `<aside>`, then it is not a `contentInfo` landmark.

### ARIA roles

| Element   | ARIA Role       | Conditions                                 |
| --------- | --------------- | ------------------------------------------ |
| `header`  | `banner`        | Only when a direct descendent of `<body>`. |
| `nav`     | `navigation`    |                                            |
| `main`    | `main`          |                                            |
| `section` | `region`        |                                            |
| `form`    | `form`          |                                            |
| `search`  | `search`        | Or form with `role="search"`               |
| `aside`   | `complementary` |                                            |
| `footer`  | `contentInfo`   | Only when a direct descendent of `<body>`. |

### Benefits

- Orient screen reader users
- Navigation: screen reader users can jump between landmarks with shortcuts