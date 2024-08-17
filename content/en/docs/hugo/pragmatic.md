---
title: "Pragmatic book"
linkTitle: "Pragmatic"
weight: 5
---

## Adding a content section

1. Create an archetype.
2. Create a layouts directory with a `list.html` and `single.html` page:
   ```
   themes/basic/layouts/presentations/
   ├── list.html
   └── single.html
   ```
3. Create an index page and any content pages:
   ```bash
   hugo new <archetype-name>/_index.md
   hugo new <archetype-name>/content-one.md
   ```



## Context 

When you work with a Hugo layout, there's a scope or context that contains the data that you want to access. The current context is is represented by a dot (`.`). This context is set to the Page context. For convenience, Hugo makes all site data available in the Page context. The Page context looks like this:

```
Context (.)
├── Site
│   └── Title
├── Title
├── Content

```
- You can access the site title in `config.toml` with `{{ .Site.Title }}`.
- `{{ .Content }}` comes from the `.md` file

## Variables

### .Date

Get the date and format it:

```go
Posted {{ .Date.Format "January 2, 2006"}}
```

### $.Variable

Prefixing a variable with a `$` tells Hugo that you want values in the global scope, not current local scope:

```go
{{- $title := printf "%s - %s" $.Page.Title $.Site.Title -}}

{{- if $.Page.IsHome -}}
    {{ $title = $.Site.Title }}
{{- end -}}
```
### .Pages

List layouts (`list.html`) let you access a collection that contains all pages related to the section you're working with:

```go
// do something with each Page in Pages
{{ range .Pages }}
    // set the context of the range function to each Page in Pages
{{ end }}
```

`.Pages` is a context-specific variable that represents a collection of pages within the current scope. The scope can vary depending on where you are in the template hierarchy. It can represent pages within a section, taxonomy, or any other subset of the site’s content.

Key Points of `.Pages`:
- Context-Sensitive: The contents of `.Pages` depend on the current context. For example, within a section template, `.Pages` will contain the pages of that section.
- Flexibility: `.Pages` can be used in different parts of the site to represent different subsets of content.


### .Site.RegularPages

Lets you access all pages in a site. You can use SQL-esque syntax to filter the pages to the ones that you need:

```go
{{ range (where .Site.RegularPages "Type" "in" "projects").ByDate.Reverse }}
    <li><a href="{{ .RelPermalink }}">{{ .Title }}</a></li>
{{ end }}
```

`.Site.RegularPages` is a site-wide variable that represents all regular content pages of the site. Regular content pages are those that are not list pages, taxonomy pages, or other special pages. This variable is used when you need to work with the entire set of content pages across the site.

Key Points of `.Site.RegularPages`:
- Site-Wide Scope: `.Site.RegularPages` includes all content pages from the entire site, regardless of their location or type.
- Excludes Non-Content Pages: It specifically excludes list pages, taxonomy pages, and other special pages.

### .Params

Lets you access custom front matter parameters:

```go
<img src="{{ .Params.image }}" alt="{{ .Params.alt_text }}">

<h3>Tech used</h3>
<ul>
    {{ range .Params.tech_used }}
        <li>{{ . }}</li>
    {{ end }}
</ul>
```
You don't have to use `.Params` to access [predefined Hugo fields](https://gohugo.io/content-management/front-matter/#fields). (you can use )

### .Page.IsHome

Checks whether the page you are on is the home page.

## Functions

### isset

Checks whether a variable has a value, but only when the variable doesn't have a default method.

The isset function is used to check whether a variable, map, or slice has been set or not. It's useful for conditional statements where you need to determine if a variable exists before proceeding with further logic.

### with

Checks whether a variable has a value, including a default value. Also rebinds the context. If the first value is not nil, then the value becomes the context:

```go
// check if there is a val set for default page var .Description
{{- with .Page.Description -}}
    // if there is, then the new context is .Page.Description
    {{ . }}
{{- else -}}
    {{ .Site.Params.description }}
{{- end -}}
```

The with function is used to simplify nested templates and to provide a scoped context. When with is used, it evaluates its argument, and if it is not nil, it executes the enclosed block with the argument as the context. If the argument is nil, it skips the block.

### getJSON

Fetches and parses JSON data from a specified URL or file. This function allows you to integrate external JSON data into your Hugo site, enabling dynamic content generation based on external sources:

```go
{{ $data := getJSON "URL or file path" }}
```

### .Site.GetPage

Pulls data from any page in the site:

```go
{{ with .Site.GetPage "/opensource.md" }}
    <h3><a href="{{ .RelPermalink }}">{{ .Title }}</a></h3>
    <p>{{ .Summary }}</p>
{{ end }}
```

### safeHTML

Use only on HTML snippets that you create.

Ensures an HTML snippet is not escaped. It allows you to mark a string as safe HTML content, meaning that Hugo will not escape the HTML tags in the string when rendering it. This is particularly useful when you want to include raw HTML in your content without Hugo escaping it, which would prevent the HTML from being rendered correctly in the browser:

```go
{{ printf $link .Rel .MediaType.Type .Permalink $title | safeHTML }}
```

### .Summary

Hugo can pull a page summary with this variable. You can control where the summary ends with `<!-- more -->`:

```md
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

<!-- more -->
```

### urlize

Encodes text content as a url

```go
<a href="/tags/{{ . | urlize}}" class="tag">{{ . }}</a>
```

## Commands

```bash
# create new theme
hugo new theme theme-name

# add new theme to config.toml
theme = "theme-name"

# create new page in /content with default archetype (do not need to create dir first)
hugo new page-name.md

# create new page in /<archetyp-name> with /archetype file
hugo new <archetype-name>/page-name.md

# delete /public and regen /public
hugo --cleanDestinationDir

# minify /public contents
hugo --cleanDestinationDir --minify
```
## Math

Perform math operations with nested inline equations:

```go
Reading time: {{ math.Round (div (countwords .Content) 200.0) }} minutes
```

## Blocks and partials

```bash
themes/basic/layouts/
├── 404.html            
├── _default            # layouts
│   ├── baseof.html     # base of every layout that you create
│   ├── list.html
│   └── single.html
├── index.html
└── partials            # all partials (components) to include in _default/<layout>.html
    ├── footer.html
    ├── header.html
    └── head.html

```

### Blocks

```go
// single.html
{{ define "main" }}
    <h2>{{ .Title }}</h2>
    {{ .Content }}
{{ end }}

// baseof.html
// pulls in the "main" block for each layout
...
<body>
    ...
    <div id="content">
    {{- block "main" . }}{{- end }}
    </div>
    ...
</body>
...
```
### Partials

Place common HTML in partials so you can reuse them as components:

```go
// {{- -}} suppresses whitespace
// partial function takes a filename and context for data
// . is the current/default context, which is the Page context
-->
{{- partial "header.html" . -}}
```

## Conditional logic


```go
// check if there is a val set for default page var .Description
{{- with .Page.Description -}}
    {{ . }}
{{- else -}}
    {{ .Site.Params.description }}
{{- end -}}
```


## Data

### Local data

Store local data in the `data/` directory, and access it with `.Site.Data.file-name.object-name`:

```json
// data file
{
  "accounts": [
    {
      "name": "Twitter",
      "url": "https://twitter.com/bphogan"
    },
    ...
  ]
}
```

```go
// in a layout
<ul>
    {{ range .Site.Data.socialmedia.accounts }}
        <li><a href="{{ .url }}">{{ .name }}</a></li>
    {{ end }}
</ul>
```

### Remote data

You can fetch remote data and iterate over it in your site. The following snippet gets the public GH repos for the url and user stored in `config.toml` params, and iterates over them to create a list:

```go
// url == gh_url/gh_user/repos
{{ $url := printf "%s/%s/repos" .Site.Params.gh_url .Site.Params.gh_user }}
// getJSON fetches and parses JSON data
{{ $repos := getJSON $url }}
<section class="oss">
    {{ range $repos }}
        <article>
            <h3><a href="{{ .html_url}}">{{ .name }}</a></h3>
            <p>{{ .description }}</p>
        </article>
    {{ end }}
```

## Layouts

Add custom layouts in the `layouts/_default/` directorys for the site, not the theme. Then, you can add the custom layout to the frontmatter with `layout`:

```yaml
---
title: "Contact"
date: 2024-06-25T23:58:40-04:00
draft: false
layout: contact
---
```

## Assets and Pipes

To use pipes, you have to create an `/assets` directory for your JS, SCSS, etc. files.

In the following snippet:
- `resources.Get` function uses `/assets` dir as its base
- `$css.RelPermalink` writes a CSS file to `/public/css` and adds the relative URL to the HTML doc
- `minify` minifies the file
- `fingerprint` creates a unique filename for the asset file so users don't cache the wrong file
- `toCSS` transpiles Sass to CSS

```go
{{ $css := resources.Get "css/style.scss" | toCSS | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```
### Sass

Use `toCSS` to transpile your Sass to CSS:

```go
{{ $css := resources.Get "css/style.scss" | toCSS | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

#### Partials

A partial is a file that you can include in the main Sass file. Partial filenames begin with an underscore:

```
/themes/.../_navbar.scss
```

Then import the file into your main Sass page where you want it to be loaded into the browser:

```scss
nav,
footer {
  background-color: #333;
  color: #fff;
  text-align: center;
}

@import "navbar";
```

## Images

For templates, you can use the [image processing](https://gohugo.io/content-management/image-processing/) functions. You can't use these in content pages, only layouts. To manipulate images, use [shortcodes](#shortcodes).

You can put images in the site `/static` directory, or the `/theme/<theme>/static` directory. The static directories are merged together.

The contents of the `/static` folder are copied to the root of the site, and the `relURL` function generates the image path relative to the page:

```html
<footer>
    <small>Copyright {{now.Format "2006"}} {{ .Site.Params.author -}}. {{- partial "social.html" . -}}</small>
    <p>Powered by<br>
        <a href="https://gohugo.io">
            <img src="{{ "hugo-logo-wide.svg" | relURL }}" alt="Hugo image" width="128" height="38">
        </a></p>
</footer>
```

## Shortcodes

Use shortcodes when you would use HTML in a Markdown document. Shortcodes are functions powered by Go's templating mechanism that you can use in md. You call them and pass them options, and they generate output. Shortcodes let you do things in your md content that you'd normally only be able to do in layouts.

Put shortcodes in `/layouts/shortcodes`, either in your theme or site with an `.html` extension:

```go
// .Get 0 gives you access to the first argument you pass to the shortcode.
// .Resize resizes the image. Pass only one value so Hugo maintains the aspect ratio.
{{ $image := $.Page.Resources.GetMatch (.Get 0)}}
{{ $smallImage := $image.Resize "1024x" }}

<figure class="post-figure">
    <a href="{{ $image.RelPermalink }}">
        {{ with $smallImage }}          // changes context so you don't have to prefix .Width and .Heigth with $smallImage
        <img src="{{ .RelPermalink }}"
             width="{{ .Width }}"
             heigth="{{ .Height }}"
             alt="{{ $.Get 1 }}" />     // fetches 2nd arg to the shortcode. Use $ because the context is changed, and you need
        {{ end }}                       // to reach outside current scope
    </a>
    <figcaption>{{ .Get 1 }}</figcaption>
</figure>
```