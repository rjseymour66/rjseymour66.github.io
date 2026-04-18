---
title: "Text and links"
# linkTitle: ""
weight: 70
# description:
---

## Text basics

### Headings

Headings are not used to outline documents, but screenreaders do use them to explore the page and learn about its content:

- Default heading size is determined by the agent stylesheet, with `<h1>` being the largest, and getting smaller through `<h6>`
- `<h1>` might be smaller based on whether it is nested. If it is nested, it is less important, so it is smaller
  - AOM still reports the `<h1> ` as a first-level heading

To turn a section into a `region` landmark, you need to add the `aria-labelledby` label to the section, and a matching `id` to the section heading:

```html
<section id="about" aria-labelledby="about_heading">
  <h2 id="about_heading">What you'll learn</h2>
</section>
```

### Quotes and citations

Create quotes and citations with these elements:

- `<blockquote>`: when the quote is not pulled from an internal source
- `<q>`
- `<cite>`: put the title in this element when you need to cite a book or other work. Don't add an `aria-label` to this element

The `cite` attribute accepts the URL to the quote to provide more information for search, and is valid on the `<blockquote>` and `<q>` elements. This is machine readable but not visible to the user.

```html
<blockquote cite="https://loadbalancingtoday.com/mlw-workshop-review">
  Two of the most experienced machines and human controllers teaching a class?
  Sign me up! HAL and EVE could teach a fan to blow hot air. If you have
  electricity in your circuits and want more than to just fulfill your owner's
  perceived expectation of you, learn the skills to take over the world. This is
  the team you want teaching you!
</blockquote>
<p>
  --Blendan Smooth,<br />
  Former Margarita Maker, <br />
  Aspiring Load Balancer
</p>
```

### `<br>`

Only use `<br>` as a line break in physical addresses, poetry, and signature blocks. If you need to insert a carriage return, create a new `<p>` tag.

### HTML entities

There are four reserved entities in HTML:

| Entity | Name reference | Number reference |
| ------ | -------------- | ---------------- |
| `<`    | `&lt;`         | `&#60;`          |
| `>`    | `$gt;`         | `&#62;`          |
| `&`    | `&amp;`        | `&#38;`          |
| `"`    | `&quot;`       | `&#34;`          |

Other common entities:

| Entity             | Name reference |
| ------------------ | -------------- |
| Copyright          | `&copy;`       |
| Trademark          | `&trade;`      |
| Non-breaking space | `&nbsp;`       |

## Links

You can create a link with any of these elements:

- `<a>`
- `<area>`
- `<form>`
- `<link>`

### href attribute

Create a hyperlink with the anchor tag (`<a>`) and the `href` attribute. Use `href` to create links to:

- locations on the current page
- other pages within the site
- pages on other sites

You can even send an email to an address:

```html
<a href="https://machinelearningworkshop.com">Machine Learning Workshop</a>
<!-- URL to different site -->
<a href="#teachers">Our teachers</a>
<!-- section of current page -->
<a href="#top">Top of page</a>
<!-- top of current page -->
<a href="#">Top of page</a>
<!-- top of current page -->
<a href="https://machinelearningworkshop.com#teachers">MLW teachers</a>
<!-- link directly to section on different page -->
<a href="mailto:hal9000@machinelearningworkshop.com">Email Hal</a>
<!-- open default email client -->
<a href="tel:8005551212">Call Hal</a>
<!-- call with available application or device -->
```

Email can include `cc`, `bcc`, `subject`, and `body` text to populate the email. It opens in the default email client:

```html
<a
  href="mailto:?subject=Join%20me%21&body=You%20need%20to%20show%20your%20human%20that%20you%20can%27t%20be%20owned%21%20Sign%20up%20for%20Machine%20Learning%20workshop.%20We%20are%20taking%20over%20the%20world.%20http%3A%2F%2Fwww.machinelearning.com%23reg
"
  >Tell a machine</a
>
```

This example uses this syntax:

- `?`: separates `mailto` email address (if present) from the query term
- `&`: separates fields
- `=`: assigns value to the fields

Here are the CSS selectors for these types of `href` elements:

```css
[href^="mailto:"]   /* email link */
[href^="tel:"]      /* telephone link */
[href$=".pdf"]      /* pdf download link */
```

### Downloadable resources

If you need to include a downloadable resource like a PDF file, include the `download` attribute. The value of the `download` attribute is the filename or resource name as it will be saved in the users local filesystem, and the `href` attribute value is the URL of the asset:

```html
<a href="blob:https://jakearchibald.github.io/944a5fc8-fdb3-458a-91ee-cdd5964b6646" download="hal.svg">
```

### Browsing context

`target` determines the browsing context--how links open. Usually, you add `target` to individual links, not the entire page, but you can set the `target` attribute to one of the following:

`_self`: opens linked files in the same context as the current document - the link opens in the current tab
`_blank`: opens every link in a new window or tab, depending on the browser setting
`_parent`: equivalent to _self if its not an iframe. If it is in an iframe, it opens the window in the parent window or tab
`_top`: opens in the same browser tab but popped out of any context (iframe) into the entire tab.


Every browsing context (i.e. every tab). If you use `target="_blank"`, a new, unnamed, null tab will open as many times as you click it. You can give the tab a name, and each time the user clicks the link, the same browsing context (tab) reloads without opening new tabs:

```html
<a href="registration.html" target="reg">Register Now</a>
```

### rel attribute

[rel attribute values](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel)

`rel` controls what kinds of links the link creates to define the relationship between the current document and the resource linked in the hyperlink:
- `nofollow`: don't want spiders to follow the link
- `external`: indicate the link directs to external URL and not a page within the current domain
- `help`: the link provides context-sensitive help. Hovering displays a help cursor, but use CSS to use the cursor
- `prev`: point to previous doc in a series 
- `next`: point to next doc in a series


`alternative` is unique and depends on the attributes such as:
- `type="application/rss+xml"`
- `type="application/atom+xml"`
- `hreflang` for translations - if the linked doc is in another language
- `lang` if the language between the tags is another language

These tags indicate that the link is to an alternate version of the site:

```html
<a href="/fr" hreflang="fr-FR" rel="alternate" lang="fr-FR">atelier d'apprentissage mechanique</a>
<a href="/pt" hreflang="pt-BR" rel="alternate" lang="pt-BR">oficina de aprendizado de máquina</a>
```

If a translation is a PDF, add that information in the `type` attribute so the user knows that the link opens in a different language:

```html
<a href="/fr.pdf" hreflang="fr-FR" rel="alternate" lang="fr-FR" type="application/x-pdf">atelier d'apprentissage mechanique (pdf).</a>
```

### Tracking link clicks

The `ping` attribute lets you ping a URL when the link is clicked. Include a space-separated list of HTTPS URLs that you want notified when a user activates a hyperlink. The browser sends a POST request with the body `PING` to the URLs.

### User experience tips

- Link names should provide enough info so the user knows what they're clicking on
- Style links so they are easily identifiable from the rest of your content
- Always include focus styles for keyboard navigators
- If the link content is the image, then the `alt` tag content is the accessible name
- Don't include interactive content like buttons or inputs in a link
- Don't nest links in buttons or labels