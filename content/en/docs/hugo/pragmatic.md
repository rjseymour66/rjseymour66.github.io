---
title: "Pragmatic book"
linkTitle: "Pragmatic"
weight: 5
---


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

## Commands

```bash
# create new theme
hugo new theme theme-name

# add new theme to config.toml
theme = "theme-name"

# create new page in /content with default archetype
hugo new page-name.md

# create new page in /<archetyp-name> with /archetype file
hugo new <archetype-name>/page-name.md

# delete /public and regen /public
hugo --cleanDestinationDir

# minify /public contents
hugo --cleanDestinationDir --minify
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

## CSS

