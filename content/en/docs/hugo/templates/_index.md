---
title: "Templates"
linkTitle: "Templates"
weight: 80
description: >
  How to add, work with, and process Templates.
---

Hugo compiles the template files in `layouts/` with the mardown content files to create HTML pages to serve to the browser.

## Go templating language

Content outside of the double braces is treated as a raw string that you can pass into the final HTML. Information in the `content/` folder is available with variables, including the following top-level variables:
- `$`: The top-level context of the template. If we are using a Page template, we can access variables with the following declarations:
  - `$.Title` or `$.Page.Title`
  - `$.Description` or `$.Page.Description`
  
  Within a shortcode, the `$` operator is shortcode-scoped.
- `site`: Provides access to data for the whole website, including configuration data. This variable is available globally. Other examples include the following:
  - `site.Pages`: Access other pages in the site.
  - `site.Taxonomies`: Taxonomy data
  - `site.Params`: The `Params` object holds all the user-defined metadata in the front matter.

    Variables in `Params` are not case-sensitive. If the frontmatter key is `subtitle`, you can access it as `{{$.Params.Subtitle}}`
- `hugo`: Provides access to the Hugo compiler. For example:
  - `hugo.IsProduction`
  - `hugo.Version`

### Existence checks

You can use an `if` statement to verify whether the value exists for a page. This lets you reuse templates across pages that might not have the same metadata or frontmatter, without fear of rendering empty strings in the HTML output.

The following examples uses `if` statements to render the page description and title when they exist. Because the title is rendered more than once, a variable is used in the existence check and then that variable is used to render the title throughout the page. The check uses the `site.Title` in the configuration as a default if there is not a local `Page.Title`:

```html
  {{if $.Description}}
  <meta name="description" content={{$.Description}}>
  {{end}}
  <link rel="stylesheet" href="./index.css">
  {{$title := $.Title}}
  {{if not $title}}
  {{$title = site.Title}}
  {{end}}

  {{if $title}}<title>{{$title}}</title>{{end}}
```
An equivalent variable assigment uses Hugo's `default` function:

```go
{{$title := default site.Title $.Title}}
```
In the previous example, the first argument to the `default` function is the default value, and the second is the value we check for existence.

`default` checks for the existence of a value, but an `if` statement checks whether the value is true or not (empty strings or 0 values do not pass the truthfulness test).

To conditionally render a value based on its truthfulness, use the `isset` function:

```go
{{if isset $.Params "subtitle"}}<h2>{{$.Params.subtitle}}</h2>{{end}}
```
The preceding example, we check if `$.Params` returns a value other than false (its true if it exists) and then conditionally render it.

### Param function pattern

The pattern of accessing a page variable and falling back to the site variable is so common that Hugo provides the built-in `$.Param` function to simplify this check. The following example provides the page subtitle and falls back to the site subtitle if the page value is not present:

```go
{{$.Param "subtitle"}}
```