---
title: "Metadata"
# linkTitle: ""
weight: 30
# description:
---

Use the `<meta>` tag for metadata that cannot be represented by the `<title>`, `<link>`, `<script>`, `<style>`, and `<base>` tags. There are many kinds of `<meta>` tags, but the most common ones are useful for SEO, socail media posting, and UX.

## Required `<meta>` tags

Originally, you defined the charset and viewport metadata with this tag:

```html
<meta http-equiv="Content-Type" content="text/html; charset=<characterset>" />
```

The `http-equiv` is short for "HTTP equivalent", which means that the meta tag is replicating what is sent in the HTTP `Content-Type` header. Developers mistyped this so much, that the specification was changed to this:

```html
<meta charset="utf-8" /> <meta name="viewport" content="width=device-width" />
```

## Official meta tags

There are two kinds of meta tags, both of which must include the `content` attribute:

- pragma directives, with the `http-equiv` attribute
- named meta types that use the `name` attribute to define the tag type

### Pragma directives

Pragma directive describe how the page should be parsed. There are [7 pragma directives](https://html.spec.whatwg.org/multipage/semantics.html#pragma-directives), but you can set them in different ways. For example, the `content-language` directive is set with the `<html lang="en">` directive.

A common one is the refresh directive, but it is a bad (and annoying) practice to include in the site:

```html
<meta
  http-equiv="refresh"
  content="60; https://machinelearningworkshop.com/regTimeout"
/>
```

The most useful is `content-security-policy`, which defines the server origins and script endpoints to guard against XSS attacks. Here is a [list of accepted values](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy) and an example:

```html
<meta http-equiv="content-security-policy" content="default-src https:" />
```

### Named meta tags

Include the `name` attribute with `content`. You should only include one of each type. Here is a list of [standard metadata names](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta/name), but the most common name meta tags include:

- `viewport`
- `description`
- `theme-color`

> Do not include `keywords`. Search engines do not use this anymore because people abused it too much for too long.

#### Description

Useful for SEO. This is what search engines display under the page's title in search results. It should be a short and accurate summary of the page's content:

```html
<meta
  name="description"
  content="Register for a machine learning workshop at our school."
/>
```

#### Robots

To request that search engines not index your site, you can use the `robots` value and include the `content=index, follow` to request indexing the site and following links on the page to crawl them (that is default behavior for the `robots` value, so it is unnecessary):

```html
<meta name="robots" content="index, follow" />
```

#### Theme color

Define a color to customize the title bar, tab bar, or other browser interfaces for supporting browsers and OSs. Useful for progressive web apps:

```html
<meta name="theme-color" content="#226DAA" />
```

## Open Graph

[OG protocol](https://ogp.me/)

Controls how social media sites displays links to your content. This lets you explictly define what these sites grab from your site. Otherwise, the sites will grab the title and description just like a search engine will without your input. For example, you can define a card with a title, image, and description for twitter, where the entire card is a hyperlink.

Requires these attributes:

- `property`: uses this format: `og:<element>`. For example, `og:title`. Refer to [OG protocol](https://ogp.me/) for more examples.
- `content`: content for the specified element

Here is an example for a facebook media card:

```html
<!-- facebook -->
<meta property="og:title" content="Machine Learning Workshop" />
<meta
  property="og:description"
  content="School for Machines Who Can't Learn Good and Want to Do Other Stuff Good Too"
/>
<meta
  property="og:image"
  content="http://www.machinelearningworkshop.com/image/all.png"
/>
<meta
  property="og:image:alt"
  content="Black and white line drawing of refrigerator, french door refrigerator, range, washer, fan, microwave, vaccuum, space heater and air conditioner"
/>
```

Twitter has its own special syntax, called [Twitter card markup](https://developer.x.com/en/docs/x-for-websites/cards/overview/markup):

```html
<!-- twitter example -->
<meta name="twitter:title" content="Machine Learning Workshop" />
<meta
  name="twitter:description"
  content="School for machines who can't learn good and want to do other stuff good too"
/>
<meta
  name="twitter:url"
  content="https://www.machinelearningworkshop.com/?src=twitter"
/>
<meta
  name="twitter:image:src"
  content="http://www.machinelearningworkshop.com/image/all.png"
/>
<meta name="twitter:image:alt" content="27 different home appliances" />
<meta name="twitter:creator" content="@estellevw" />
<meta name="twitter:site" content="@perfmattersconf" />
```

## Application icons and titles

Add icons for someone that favorites your app and puts it on the homescreen. These are called "startup images":

```html
<link
  rel="apple-touch-startup-image"
  href="icons/ios-portrait.png"
  media="orientation: portrait"
/>
<link
  rel="apple-touch-startup-image"
  href="icons/ios-landscape.png"
  media="orientation: landscape"
/>
```

If you want the icon to have a shortened name on the homescreen, define it with these tags:

```html
<meta name="apple-mobile-web-app-title" content="MLW" />
<meta name="application-name" content="MLW" />
```

## Web-app capable

If your site uses a minimal UI and doesn't require the browser's back button, you can declare it as `app-capable`:

```html
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="mobile-web-app-capable" content="yes" />
```

## Web manifest file

To prevent a long list of `<meta>` tags, you can create a [web manifest file](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest) and link to it with a relationship link:

```html
<link rel="manifest" href="/mlw.webmanifest" />
```