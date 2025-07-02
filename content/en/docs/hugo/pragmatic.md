---
title: "Pragmatic book"
linkTitle: "Pragmatic"
weight: 5
---

## Big ideas

> The layout structure changed with Hugo 1.46. Here is a breakdown: https://gohugo.io/templates/new-templatesystem-overview/. Notable changes include the following:
> - `layouts/_default/` directory is removed, and all layouts files go in the `layouts/`.
> - `layouts/index.html` is now `layouts/home.html`

- An archetype is a content template for a markdown page. A layout page is an HTML page that is generated from a markdown page. You need to associate a layout page with markdown pages generated from an archetype. For example, `layouts/<file>.html` is used for all pages.
  
  The home page is different--it generates `layouts/index.html` with content in `content/_index.md`.
- `layouts/baseof.html` is the skeleton for all layout files--the "base of" all files. It contains partials that define the head, header, main, etc.
- The default context for a layout page is the Page context (`.`).
- Partial functions take a filename and a context.

## Ch 1 Creating your site

Generate a website project with this command. It will create a directory using the provided `<site-name>`, so you might have to move all generated contents into the current working directory if you already have a git repo cloned:

```bash
hugo new site site-name       # create website
mv site-name/* .              # move generated files to pwd
rmdir site-name               # delete empty dir
```

### Directory structure

https://gohugo.io/getting-started/directory-structure/

```bash
site-name
├── archetypes              # md templates for your content types
│   └── default.md
├── assets                  # global resources like CSS, Sass, JS, etc
├── content                 # md files that comprise the site content
├── data                    # JSON, YAML, XML, etc files that you can extract to populate your site
├── hugo.toml               # site config file
├── i18n                    # translation tables for multilingual sites
├── layouts                 # template files that define look and feel of site
├── static                  # static assets (CSS, images, JS, ect) copied to the public/ dir when you build your site
└── themes                  # themes that you download or create
```

### Building the home page

Each type of page that you have in your site will have its own layout page. The home page has its own layout because you likely want your home page to be different than the rest of the site.

Add all layouts in the `layouts/` directory. Here, we add the home page:

```bash
touch layouts/home.html
```
Here is the example layout:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{ .Site.Title }}</title>
  </head>
  <body>
    <h1>{{ .Site.Title }}</h1>      <!-- Site context -->

    {{ .Content }}                  <!-- Page context -->
  </body>
</html>
```

The `{{ .Content }}` for the home page comes from a special markdown file, the `content/_index.md` file. To to complete the home page, create that file and enter content.

### Creating content with archetypes

Hugo has commands that create content pages with placeholder content. When you created the site with `hugo new site site-name`, Hugo created the `archetypes/default.md` page with this content:

```md
+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
+++
```

The `hugo new` command uses this `default.md` page as its template:

```bash
hugo new pagename.md
```

### Archetypes and layouts

When you run the `hugo new pagename.md` command, you generate a markdown file with the placeholder text in `archetypes/default.md`. This creates the content page, but Hugo won't generate an HTML page for this content unless there is a corresponding page in `layouts/`. The default single page layout that every content page uses is in `layouts/_default/`. This page can have any name at all, but here we name it `single.html` because it is the default layout for a single page of content:

```bash
touch layouts/single.html
```

You can copy the skeleton from the `content/_index.html` page and then make updates for the single template.

### Layout contexts

When you create a layout page, you add go templating (`{{ .Content }}`) in the HTML. This data is replaced with site or page data when you build the website. Each page has a content context which determines the data you can access from the page. Layout pages use the Page context, represented by a dot (`.`). Here is a simple representation:

```
Context (.)
├── Site
│   └── Title
├── Title
├── Content
```

The `Site` data comes from the `hugo.toml` configuration file, and the Page context data comes from the markdown file associated with the page.


### Generate public/ folder

`hugo server` generates content in memory. To write contents to disk, run `hugo` with no arguments:

```bash
hugo
```
This generates your static site in the `public/` folder. This is the content that you upload to a CDN to publish the site (make it _public_...).

When you remove or rename pages, you need to clean the `public/` folder. You can either delete it from the project, or you can add the `--cleanDestinationDir` flag when you build the site. You can also minify the files with the `--minify` flag:

```bash
hugo --cleanDestinationDir --minify
```

## Ch 2 Building a theme


Create a new theme with this command. It will generate theme directories in `/themes/<theme-name>`:

```bash
hugo new theme theme-name
```
To use a theme in your site, add it to the config file:

```toml
theme = "theme-name"
```

A basic theme needs only these files:

```bash
layouts/
├── default/
│   └── list.html       
│   └── single.html     # default archetype
├── index.html          # home page
```
### Partials

`layouts/baseof.html` is the skeleton for all layout files--the "base of" all files. It contains partials that define the head, header, main, etc. A partial is a file that contains commonly reused parts of the layout, such as the head, header, etc. 

To build a partial, add an `.html` file to `layouts/_partials/`, and add a partial function reference where you want Hugo to generate the output:

```go
{{- partial "filename.html" . -}}
```
A partial function takes a filename and a context. The default context for a layout page is the Page context. Here, we grab the Page context with the dot (`.`), which represents the current context.

The dashes before and after the double curly braces (`-`) tells Hugo to remove any whitespace in the output files.

#### Creating a partial

1. Create the partial HTML file in `theme/theme-name/layouts/_partials`:
   ```bash
   touch theme/theme-name/layouts/_partials/nav.html
   ```
2. Add your HTML and any Go templating.
3. Go where you want to use the partial, and add the partial function with Go templates:
   ```go
   {{ partial "nav.html" . }}
   ```

### Content blocks

A content block is defined with the `block` function. Here is an example from the `baseof.html` page:

```html
...
<main>
    {{ block "main" . }}{{ end }}
</main>
...
```
This line tells Hugo to find the main block in the content file, and generate it here. This means that you don't have to duplicate your head, header, footer, and other boilerplate HTML in your layout files.

So, `baseof.html` has a `<main>` element with a `main` block that takes the current Page context. This function renders content placed in layout pages. It is a placeholder for the `main` blocks defined in your layout pages. `single.html` defines only a `main` content block, so Hugo performs the following:
1. Finds `content/*.md` files that use the `single.html` layout, and grabs the content in the `.md` file.
2. Formats the content using the `main` block in `single.html`.
3. Generates a page using `baseof.html` as the template, and the `single.html` markdown content as the `main` block.

Here is the `single.html` layout file that defines its main block:

```go
{{ define "main" }}         // start main block      
  <h2>{{ .Title }}</h2>     // h2 starts the main block
  
  {{ .Content }}            // Page content is pulled in
  
{{ end }}                   // end main block
```
So, Hugo generates HTML with the page title and content in `content/file.md`, then places it in the `<main>` block of `baseof.html`.


### Adding CSS

If you're using Hugo Pipes to process your asset files (minify, bundle, etc), place them in the `theme-name/assets/` directory. Otherwise, place them in the `theme/theme-name/static/` directory.

Place all theme-specific styles in the `theme/theme-name/` directory tree. Site-specific images and files should go in your site directories.

After you define your theme styles in `theme/theme-name/static/css/style.css`, load them into your site in the `layouts/_partials/head.html` partial with this tag:

```html
<link rel="stylesheet" href="{{ "css/style.css" | relURL }}">
```
Here, we pipe the static directory and filename to the `relURL` function. This function takes the argument and transforms it into a relative URL from the current page. So, no matter where you are in your website, the CSS link always works.

To create an absolute URL that includes the site domain, use `absURL`.

## Ch 3 Adding Content Sections

Archetypes are content templates. You can create a new archetype in your site or theme, then generate a markdown page using the template with this command:

```bash
hugo new <archetype>/filename.md
```
Hugo will create an `<archetype>` directory if it doesn't exist, and then create the `filename.md` file using the archetype as a template. For example, if you want to create a blog post, you create an archetype titled `posts.md`, then run this command to create your first post:

```bash
hugo new posts/first-post.md
```
Hugo first looks for the `posts` archetype in your site, then in the theme, and then it falls back to the `default.md` archetypes.

### Create a content section

The requirements for a new content section. For example, to create a new section called "Tutorials":
1. Create a new archetype named `tutorials.md`:
   ```bash
   touch archetypes/tutorials.md
   ```
2. Create a single page template and section (list) template:
   ```bash
   mkdir theme/docsite/layouts/tutorials
   touch theme/docsite/layouts/tutorials/{single.html,section.html}
   ```
3. Create a content page:
   ```bash
   hugo new tutorials/filename.md
   ```
4. Optionally, create an `_index.md` page to add custom content to tutorial section's landing page:
   ```bash
   touch content/tutorials/_index.md
   ```

### Section layouts

Each directory, or section, in your `content/` directory can have a default page that lists the content of the directory. This used to be called the `list.html` page, but it was renamed to `section.html`.

To loop over all the pages in the current directory, use the `range` function with the `Pages` collection:

```go
{{ define "main" }}
  <h1>{{ .Title }}</h1>
  {{ .Content }}
  {{ range .Pages }}
    <h2><a href="{{ .RelPermalink }}">{{ .LinkTitle }}</a></h2>
    {{ .Summary }}
  {{ end }}
{{ end }}
```

The `Pages` collection contains all the pages in the current section. `range` iterates over the collection, which lets you access `Page` properties like `Title` and `LinkTitle`. (`LinkTitle` returns the `linkTitle` frontmatter property or the page title as a fallback.) `RelPermalink` constructs a relative link from the site root (`PermaLink` returns the absolute URL, including the site root).

#### Custom section layouts

To create a custom section layout for a specific directory tree, add a directory in `layouts/` with the same name as the content directory, and then add a `section.html` file:

```bash
docsite
│
...
├── content
│   ...
│   ├── projects                    # matches themes/docsite/layouts/projects
│   │   ├── awesomeco.md
│   │   ├── _index.md               # uses themes/docsite/layouts/projects/section.html
│   │   └── jabberwocky.md
│   ...
...
└── themes
    └── docsite
        ...
        ├── layouts
        │   ...
        │   ├── projects            # matches content/projects
        │   │   ├── section.html    # layout for content/project/<list>
        │   │   └── single.html
        │   ├── section.html        # layout for content/*/<list>
        │   ...
        ... 
```

### Section page content

If you want to add custom content to this page, add a `{{ .Content }}` block to `section.html`, and create an `_index.md` file in the directory and add content.

## Left-hand TOC

You can loop through all pages in the site or all pages in the section with the `Site.RegularPages` variable.

This template page creates two sections and lays them side-by-side with flexbox:

```html
{{ define "main" }}
    <div class="project-container">
        <section class="project-list">
            <h2>Projects</h2>
            <ul>
                {{ range .Site.RegularPages }}
                    <li><a href="{{ .RelPermalink }}">{{ .Title }}</a></li>
                {{ end }}
            </ul>
        </section>
        <section class="project">
                <h2>{{ .Title }}</h2>
                {{ .Content }}
        </section>
    </div>
{{ end }}
```

Here are the styles that lay each section side-by-side with flexbox:

```css
.project-container {
    display: flex;
}
.project-container .project-list {
    width: 20%;
}
.project-container .project {
    flex: 1;
}
```

The `<ul>` contains a `range` function that iterates over a collection of Pages. Here, you loop over all pages in the site:

```go
{{ range .Site.RegularPages }}
    <li><a href="{{ .RelPermalink }}">{{ .Title }}</a></li>
{{ end }}
```

Or you use the [where](https://gohugo.io/functions/collections/where/) function to specify a specifiy collection of pages. Here, you look in the "projects" folder for the pages:

```go
(where .Site.RegularPages "Type" "in" "projects")
```

`Type` is a built-in Hugo page property that defaults to the first directory name under content/ where the page lives.

Here is another example that displays the most recently added file in the projects directory:

```go
{{ $recent := first 1 (sort (where .Site.RegularPages "Section" "projects") "Date" "desc") }}
    {{ with index $recent 0 }}
        <h2><a href="{{ .RelPermalink }}">{{ .Title }}</a></h2>
{{ end }}
```


## Default page types

| Page Type                   | Description                                                                            | Default Layout Used            |
| --------------------------- | -------------------------------------------------------------------------------------- | ------------------------------ |
| `single`                    | A single content file (e.g. `content/posts/my-post.md`)                                | `layouts/single.html`          |
| `section` (formerly `list`) | A list of content under a section (e.g. `content/posts/_index.md` or `content/posts/`) | `layouts/section.html`         |
| `home`                      | The homepage (`/`)                                                                     | `layouts/index.html`           |
| `term`                      | A list of all pages for a taxonomy term (e.g. `/tags/foo/`)                            | `layouts/terms.html`           |
| `taxonomy`                  | A list of all terms under a taxonomy (e.g. `/tags/`)                                   | `layouts/taxonomy.html`        |
| `404`                       | The 404 error page                                                                     | `layouts/404.html`             |
| `rss`                       | RSS feed for a section or taxonomy                                                     | `layouts/rss.xml` or `rss.xml` |
| `robots.txt`                | Robots.txt page                                                                        | `layouts/robots.txt`           |
| `sitemap.xml`               | Sitemap                                                                                | `layouts/sitemap.xml`          |


## Important layout files

| File                    | Used For                     |
| ----------------------- | ---------------------------- |
| `layouts/single.html`   | Default for regular content  |
| `layouts/section.html`  | Sections and list pages      |
| `layouts/index.html`    | Homepage                     |
| `layouts/terms.html`    | Pages for each taxonomy term |
| `layouts/taxonomy.html` | Overview of taxonomy terms   |
| `layouts/404.html`      | Custom 404 page              |


## Ch 4 Working with data

> Info about Google Analytics is on pg 36.

### Site configuration data

Access data in `hugo.toml` with the `.Site` object. Hugo has many [configuration settings](https://gohugo.io/configuration/all/) that you can add. Here, we add a [params](https://gohugo.io/configuration/params/) section with the author and description field:

```toml
[params]
  author = "Arthur Gname"
  description = "My portfolio site"
```

We want to add this info to the `<head>` on all pages, so we go to `themes/docsite/layouts/_partials/head/head.html` and add the following:

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<meta name="author" content="{{ .Site.Params.author }}">
<meta name="description" content="{{ .Site.Params.description }}">
...
```

### Front matter data

You can access a page's front matter in the layout. All front matter fields are added to the `.Params` collection, so use that to access the value. If you are uses a native [parameter](https://gohugo.io/content-management/front-matter/#parameters), you don't have to use `.Params`:

```toml
+++
title = "{{ replace .Name "-" " " | title }}"
draft = false
image = "https://placehold.co/640x150"
alt_text = "{{ replace .Name "-" " " | title }} screenshot"
summary = "Summary of the {{ replace .Name "-" " " | title }} project"
tech_used =  ["Javascript", "CSS", "HTML"]
description = "this is it"
+++
```

```go
<section class="project">
        <h2>{{ .Title }}</h2>
        {{ .Content }}
        <img src="{{ .Params.image }}" alt="{{ .Params.alt_text }}">

        <h3>Tech used</h3>
        <ul>
            {{ range .Params.tech_used }}
                <li>{{ . }}</li>
            {{ end }}
        </ul>
</section>
```

The `{{ .Summary }}` example here is in `section.html`, but it works because the context of each `range` loop is a `.Page`:

```go
{{ range .Pages }}
<section class="project">
    <h3><a href="{{ .RelPermalink }}">{{ .Title }}</a></h3>
    <p>{{ .Summary }}</p>
</section>
{{ end }}
```

### Conditionally displaying data

#### with

When you need to conditionally display values, use `with`. `with` rebinds the context to the value that you pass it. This is great when you have values that are defined but are empty. For example, a default value like `description`. If you don't give it a value, it is avaialable but empty.

To demonstrate, this `with` expression changes the context from the outer context (such as `.Page`) and changes it to the `Page.Description` context. So, when the following line uses only the current context (`{{ . }}`), it prints the `Page.Description` value. If that value is empty, it executes the `else` block:

```go
{{- with .Page.Description -}}          // rebind to .Page.Description context
    {{ . }}                             // . refers to .Page.Description. If its empty, skip
{{- else -}}                            // Otherwise, use the .Site.Params.description
    {{ .Site.Params.description }}
{{- end -}}">
```
So, if you have a `description` value in the page frontmatter, it uses that value. Otherwise, it uses the description in the `hugo.toml` config file.

#### IsHome

Here, we use an if/else clause to change the page title, depending on whether we are on the home page:

```go
<title>
    {{- if .Page.IsHome -}}
        {{ .Site.Title }}
    {{- else -}}
        {{ .Title }} - {{ .Site.Title }}
    {{- end -}}
</title>
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

Image processing can increase build times. To make it shorter, Hugo caches any converted images and saves them in the `/resources` folder. You can clean up these images with this command:

```bash
hugo -gc
```

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

## Javascript

You can combine all scripts into a singl eminified file and fingerprint the file to protect against incorrect cached files.

Put your JS files in `themes/<theme-name>/assets/js`, and add them to your `themes/<theme-name>/layouts/_default/filename.html` file like this:

```go
    // no webpack
    {{ $lunr := resources.Get "js/lunr.js" }}       // <script src="//unpkg.com/lunr@2.3.6/lunr.js"></script>
    {{ $axios := resources.Get "js/axios.js" }}     // <script src="//unpkg.com/axios@0.19.0/dist/axios.js"></script>
    {{ $search := resources.Get "js/search.js" }}   // <script src="{{ "js/search.js" | relURL }}"></script>
    {{ $libs := slice $lunr $axios $search }}       // create a slice
    {{ $js := $libs | resources.Concat "js/app.js" | minify | fingerprint }} // combine into single file, minify, then fingerprint
    <script src="{{{ $js.RelPermalink }}</script>    
```

### Webpack and npm

Webpack manages and builds frontend applications. Its powered by Node.js, so you have access to npm for package management and automation.

To use Webpack with Hugo, you have to do the following:
1. Install the dependencies
2. Create a `package.json` file with scripts that run Webpack and hugo
3. Create a `webpack.config.js` file that tells Webpack where to find your source JS files and build the output.
4. Add a path to your Webpack JS output file in your HTML.

#### Install dependencies

```bash
# Install webpack and its CLI
npm install --save-dev webpack webpack-cli

# Install dependencies -- you don't have to use 
# <script src="<dep[1,2,...]>" in HTML files
npm install --save <dep1> <dep2> ...

# Run multiple tasks at once or parallel
npm install --save-dev npm-run-all
```

#### Create package.json

Sample `package.json` file:

```json
{
  "name": "portfolio",
  "version": "1.0.0",
  "description": "My portfolio",
  "private": true,
  "scripts": {
    "build": "npm-run-all webpack hugo-build",  // run webpack and hugo server sequentially
    "hugo-build": "hugo --cleanDestinationDir", // build hugo
    "hugo-server": "hugo server --disableFastRender",   // run dev server, nothing is cached
    "webpack": "webpack",   // runs webpack
    "webpack-watch": "webpack --watch", // webpack watches changes for hugo and generates new .js file (run in new terminal)
    "dev": "npm-run-all webpack --parallel webpack-watch hugo-server"
  },
  "devDependencies": {
    "npm-run-all": "^4.1.5",
    "webpack": "^5.93.0",
    "webpack-cli": "^5.1.4"
  },
  "dependencies": {
    "axios": "^1.7.4",
    "lunr": "^2.3.9"
  }
}
```

#### Create webpack.config.js

Add `webpack.config.js`:

```js
const path = require('path');

module.exports = {
    entry: './themes/basic/assets/js/index.js', // looks for index.js
    output: {   // generates output in themes/basic/assets/js/app.js
        filename: 'app.js',
        path: path.resolve(__dirname, 'themes', 'basic', 'assets', 'js')
    }
};
```
#### Integrate JS output file in HTML file

Integrate the file that Webpack generates into the layout:

```html
{{ $js := resources.Get "js/app.js" | minify | fingerprint }}
    <script src="{{ $js.RelPermalink }}"</script>
```