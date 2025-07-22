---
title: "Pragmatic book"
linkTitle: "Pragmatic"
weight: 5
---

## Debugging

### Inspect object

```go
<pre>{{ debug.Dump .Page }}</pre>
```

### Inspect variables

```go
<pre>{{ . | jsonify (dict "indent" "  ") | safeHTML }}</pre>
```
### Print specific values

```go
{{ printf "Page title: %s" .Title }}
{{ printf "Permalink: %s" .Permalink }}
```

### Stop and log errors

```go
{{ errorf "Missing param: %s" .Params.missingThing }}
```
### Verbose build

```bash
hugo --verbose
```
### Highlight undefined variables

```go
{{ if not .Params.title }}
  <strong style="color:red">Missing title param</strong>
{{ end }}
```
### Print current page context

```go
<pre>{{ .File | jsonify (dict "indent" "  ") | safeHTML }}</pre>
```
### Inspect .Site.Menus and .Children

```go
<pre>{{ site.Menus.main | jsonify (dict "indent" "  ") | safeHTML }}</pre>
```
### Debug layouts 

```bash
hugo --debug
```

OR

```html
<p>Layout: {{ .Layout }}</p>
<p>Type: {{ .Type }}</p>
<p>Kind: {{ .Kind }}</p>
```
### Dump page variables

```go
<pre>{{ . | printf "%#v" | safeHTML }}</pre>
```

### Conditional logic to prevent noise

```go
{{ with .Params.tags }}
  {{ range . }}
    {{ printf "Tag: %s" . }}
  {{ end }}
{{ else }}
  <p>No tags found.</p>
{{ end }}
```

### Rerender on changes (template debugging)

```bash
hugo server --disableFastRender
```

## Big ideas

> The layout structure changed with Hugo 1.46. Here is a breakdown: https://gohugo.io/templates/new-templatesystem-overview/. Notable changes include the following:
> - `layouts/_default/` directory is removed, and all layouts files go in the `layouts/`.
> - `layouts/index.html` is now `layouts/home.html`

- An archetype is a content template for a markdown page. A layout page is an HTML page that is generated from a markdown page. You need to associate a layout page with markdown pages generated from an archetype. For example, `layouts/<file>.html` is used for all pages.
  
  The home page is different--it generates `layouts/index.html` with content in `content/_index.md`.
- `layouts/baseof.html` is the skeleton for all layout files--the "base of" all files. It contains partials that define the head, header, main, etc.
- The default context for a layout page is the Page context (`.`).
- Partial functions take a filename and a context.

## Writing functions

Here are the general steps to create a function using your content and Hugo built-ins. Here are some general tips:

- `printf` or `scratch` for formatting and intermediate variables
- `with`, `if`, and `range` to conditionally render blocks
- `.Site` for global data, `.Params` for front matter

1. Start with the data:
   - `.Content`: rendered HTML
   - `.RawContent`: raw markdown source
   - `.Title`, `.Date`, `.Params`: common page fields

2. Apply functions with pipeline syntax:
   ```go
   {{ $words := countwords .Content }}
   {{ $readingTime := div $words 250 }}
   ```
3. Use Hugo built-in functions:
   - `countwords`: counts words
   - `len`: gets length of lists/maps/strings
   - `add`, `div`, `mul`, `sub`: math
   - `math.Round`, `math.Floor`, `math.Ceil`
   - `time`, `now`, `dateFormat`
   - `where`, `index`, `delimit`, `replace`

For example, here we show the number of days since the page was last updated:

```go
{{ $days := div (sub now.Unix .Lastmod.Unix) 86400 }}
<p>Updated {{ math.Floor $days }} days ago</p>
```

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

> Hugo first looks in your site's `layout/` directory for a layout, then the theme's `layout/` directory.

When you run the `hugo new pagename.md` command, you generate a markdown file with the placeholder text in `archetypes/default.md`. This creates the content page, but Hugo won't generate an HTML page for this content unless there is a corresponding page in `layouts/`. The default single page layout that every content page uses is in `layouts/_default/`. This page can have any name at all, but here we name it `single.html` because it is the default layout for a single page of content:

```bash
touch layouts/single.html
```

You can copy the skeleton from the `content/_index.html` page and then make updates for the single template.

#### Specifying a layout

Use the `layout` front matter property to specify which layout you want to use for a page. This example specifies the `layouts/contact.html` layout:

```toml
+++
date = '2025-06-29T09:05:04-04:00'
draft = false
title = 'Contact'
layout = 'contact'
+++
```

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
Hugo creates an `<archetype>` directory if it doesn't exist, and then create the `filename.md` file using the archetype as a template. For example, if you want to create a blog post, you create an archetype titled `posts.md`, then run this command to create your first post:

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

### Config file fallback data

This example uses the `keywords = []...` front matter field. Otherwise, it uses the `[keywords]` section in `hugo.toml`. If there are no keywords in either, nothing is displayed. Notice that the front matter field has a captial 'K', and the config file field is lowercase. Also notice that you have a `with else` clause, not just an `else if`:
```go
{{- with .Page.Keywords -}}
    <meta name="keywords" content="{{ delimit . ", " }}">
{{- else with .Site.Params.keywords -}}
    <meta name="keywords" content="{{ delimit . ", " }}">
{{- end -}}
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

### Local data files

If you want to use local data files, you need to do store the data file in the `data/` directory for your site:
```bash
docsite
├── archetypes
...
├── assets
├── content
...
├── data
│   └── socialmedia.json
```

Here are the contents of `socialmedia.json`:

```json
{
  "accounts": [
    {
      "name": "Twitter",
      "url": "https://twitter.com/bphogan"
    },
    {
      "name": "LinkedIn",
      "url": "https://linkedin.com/bphogan"
    }
  ]
}
```

Then, use the data in a layout. Here, we create a `docsite/layouts/contact.html` file that ranges over the file data:

```go
{{ define "main" }}

...
    <ul>
        {{ range .Site.Data.socialmedia.accounts }}
            <li><a href="{{ .url }}">{{ .name }}</a></li>
        {{ end }}
    </ul>

{{ end }}
```

- Access the file with `.Site.Data.<filename>.<obj-name>`. Do not include the file extension in `<filename>`.
- `range` changes the context to the `accounts` object, so you can access each object property with a dot and its field name

### Remote data sources

Every time you build the site, Hugo makes a call to your remote resource and populates the page. If you don't want to make the API call, you can download the data and store it in your `data/` directory and access it locally.

This example retrieves all public repos for a GitHub username and populates a list with them.

1. First, define GitHub user values in `hugo.toml`:

```toml
[params]
  gh_url = "https://api.github.com/users"
  gh_user = "rjseymour66"
```

2. Next, create the layout file:
   ```bash
   touch themes/docsite/layouts/opensource.html
   ```
3. In the layout, retrieve the repo information and then range over the JSON to populate the list. See the offical docs for [Remote resource](https://gohugo.io/functions/transform/unmarshal/#remote-resource).
   1. Instantiate an empty dictionary to hold the list of GH repos
   2. Construct the GH URL with site configuration parameters
   3. Fetch remove JSON data with `resources.GetRemote $url`. Use `try` for try-catch error handling, and `with` to skip this step if the fetch fails.
      `try` returns `TryValue`, which has the `.Err` and `.Value` methods. [See the docs](https://gohugo.io/functions/go-template/try/).
   4. If there is an error in the request, print it and fail. `errorf` calls log.Fatal to kill the build.
   5. If there is a value, take the .Value, pipe it to `transform.Unmarshal`, then assign it to the `$repos` variable. [transform.Unmarshal](https://gohugo.io/functions/transform/unmarshal/) parses serialized data.
   6. If there is no .Err or .Value, then something went wrong and we log it.
   7. Loop through the fetched data stored in `$repos`.
   ```html
   {{ define "main" }}

       <h2>{{ .Title }}</h2>
   
        {{ .Content }}
   (1)  {{ $repos := dict }}
   (2)  {{ $url := printf "%s/%s/repos" .Site.Params.gh_url .Site.Params.gh_user }}
   (3)  {{ with try (resources.GetRemote $url) }}
   (4)      {{ with .Err }}
                {{ errorf "%s" . }}
   (5)      {{ else with .Value }}
                {{ $repos = . | transform.Unmarshal }}
   (6)      {{ else }}
                {{ errorf "Unable to get remote resource %q" $url }}
            {{ end }}
        {{ end }}
   
   (7)  <section class="oss">
           {{ range $repos }}
               <article>
                   <h3><a href="{{ .html_url }}">{{ .name }}</a></h3>
                   <p>{{ .description }}</p>
               </article>
           {{ end }}
       </section>
   {{ end }}
   ```
4. Generate the content file:
   ```bash
   hugo new opensource.md
   ```
5. Specify the layout in the frontmatter:
   ```toml
   +++
   date = '2025-07-03T09:47:54-04:00'
   draft = false
   title = 'Open Source Software'
   layout = 'opensource'
   +++
   ```


### RSS feeds

An RSS feed is a live list of content on a site that includes the latest pages, and the title, link, summary, and publish date. Hugo automatically generates an RSS feed for your site at `<site-root>/index.xml` and for any content directory that has child pages. Here is an example from the site root `index.xml` file:

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>My Portfolio Site</title>
    <link>http://localhost:38873/</link>
    <description>Recent content on My Portfolio Site</description>
    <generator>Hugo</generator>
    <language>en-us</language>
    <lastBuildDate>Thu, 03 Jul 2025 09:47:54 -0400</lastBuildDate>
    <atom:link href="http://localhost:38873/index.xml" rel="self" type="application/rss+xml" />
    <item>
      <title>Open Source Software</title>
      <link>http://localhost:38873/opensource/</link>
      <pubDate>Thu, 03 Jul 2025 09:47:54 -0400</pubDate>
      <guid>http://localhost:38873/opensource/</guid>
      <description></description>
    </item>
    ...
```

You need to make your feed discoverable with a `meta` tag in the site's header. The tag needs to look like this:

```html
<link rel="alternative" type="application/rss+xml" href="http://example.com/feed">
```

Here is the Hugo template to add it in `themes/layouts/_partials/head.html`. Read the [AlternativeOutputFormats docs](https://gohugo.io/methods/page/alternativeoutputformats/), too:

1. Loops through all alternative output formats enabled for the page.
2. Builds a format string for the final link element.
3. Builds the `title` attribute for the link element.
4. If this is the homepage, the link's `title` attribute should be the title defined in `hugo.toml`. Because the scope of the range functin is `AlternativeOutputFormats`, you must add the `$` to use the global scope for access to the config file.
5. Fills in values of the `$link` format string:
   1. Gets the `rel` value of hte output format
   2. Gets the media type of the output format and its subtype.
   3. Gets the absolute URL of the generated RSS page
   4. Adds the title

```go
(1)  {{ range .AlternativeOutputFormats -}}

(2)      {{- $link := `<link rel="%s" type="%s" href="%s" title="%s">` -}}
(3)      {{- $title := printf "%s - %s" $.Page.Title $.Site.Title -}}

(4)      {{- if $.Page.IsHome -}}
            {{ $title = $.Site.Title }}
         {{- end -}}

(5)      {{ printf $link .Rel .MediaType.Type .Permalink $title | safeHTML }}

    {{- end }}
```
Here is the generated HTML:

```html
<link rel="alternate" type="application/rss+xml" href="http://localhost:38873/presentations/index.xml" title="Presentations - My Portfolio Site">
```
### Rendering content as JSON

Hugo can create a JSON API that you can consume from other applications. If you looped through the [alternative formats](#rss-feeds), the JSON file will be included in the header for the applicable `<section>/section.html` page:

1. Add the outputs you want for a section page to `hugo.toml`:

   ```toml
   [outputs]
   section = ["HTML", "JSON", "RSS"]
   ```

2. Create a JSON template in the layout directory. The presence of this file tells Hugo to generate the JSON output for this section:

   ```bash
   touch themes/layouts/projects/section.json
   ```

3. Add the Go templating that defines the JSON structure:
   1. Begin the JSON object
   2. Loop over all regular pages in the projects directory, and assign these:
      1. loop iteration number to `$index`
      2. Page object to `$page`
   3. If `$index` is non-zero (not the first element), add a comma to make this valid JSON
   4. Get the absolute URL and format it for JSON
   5. Get the page title and format it for JSON
   
   
   ```go
   {
   (1)    "projects": [
   (2)        {{- range $index, $page := (where .Site.RegularPages "Type" "in" "projects") }}
   (3)        {{- if $index -}} , {{- end }}
              {
   (4)            "url": {{ .Permalink | jsonify }},
   (5)            "title": {{ .Title | jsonify }}
              }
              {{- end }}
          ]
   }
   ```

## Ch 5 Adding a Blog

### .Summary content

Add `<!-- more -->` to the file to indicate the end of the page summary. Hugo will grab the text before `<!-- more -->` and use it as the `.Page.Summary` value:

```md
Lorem ipsum dolor sit amet consectetur adipiscing elit. Quisque faucibus ex sapien vitae pellentesque sem placerat. In id cursus mi pretium tellus duis convallis. Tempus leo eu aenean sed diam urna tempor.

<!-- more -->
...
```

### Blog post format

Here is a simple format for a blog post. It uses site configuration info to populate some content, and it uses Hugo functions to calculate the amount of time that it takes to read the blog (assuming a person reads 200 wpm):

```go
{{ define "main" }}
<article class="post">
    <header>
        <h2>{{ .Title }}</h2>
        <p>By {{ .Params.Author }}</p>
        <p>Posted {{ .Date.Format "January 2, 2006" }}</p>
        <p>Reading time: {{ math.Round (div (countwords .Content) 200.0) }} minutes.</p>
    </header>
    <section class="body">
        {{ .Content }}
    </section>
</article>
{{ end }}
```
### Syntax highlighting

Hugo supports the Chroma highlighting library and the Pygments syntax highlighter. Here are the setup steps:
1. Tell Hugo that you want to use Pygments-style classes to highlight your code. Add this to `hugo.toml`:
   ```toml
   pygmentsUseClasses = true
   ```
2. Generate a stylesheet to highlight your code:
   ```bash
   hugo gen chromastyles --style=github > syntax.css
   ```
3. Move `syntax.css` to your CSS directory.
4. Add the generated `syntax.css` stylesheet to your `head.html` partial:
   ```html
   <link rel="stylesheet" href="{{ "css/syntax.css" | relURL }}">
   ```
### Custom taxonomy page

Hugo gives you a default taxonomy page named `taxonomy.html`. This is the default template for the `site/categories` and `site/tags` pages. So, if you have a `categories` front matter, the `site/categories` page is populated with those values usng the `taxonomy.html` template.

This updated page lists all tag or category values and their number of occurrences:

```go
{{ define "main" }}

    <h2>{{ .Title }}</h2>

    {{ .Content }}

    {{ range .Data.Terms.Alphabetical }}                        // .Data.Terms in alpha order
    <p class="tag">
        <a href="{{ .Page.Permalink }}">{{ .Page.Title }}</a>   // Title is link
        <span class="count">{{ .Count }}</span>                 // Number of occurrences of .Data.Terms
    </p>
    {{ end }}

{{ end }}
```

To generate default content for either data type, create a directory and `_index.md` file:

```bash
hugo new tags/_index.md
hugo new categories/_index.md
```



### Disabling taxonomies

Hugo creates default tags and categories pages, but you can disable one or both of them. You have to add info to `hugo.toml` that tells Hugo to exclude one or the other.

```toml
# disable tags
[taxonomies]
  category = "categories"

# disable categories
[taxonomies]
  tag = "tags"

# disable both
disableKinds= ["taxonomy", "taxonomyTerm"]
```

### Adding tags to a page

Here, we add `tags` from the front matter to the `themes/theme-name/layouts/posts/single.html` layout. It loops through the `Params.tags` values, and uses the `urlize` function to encode the tag as a URL-safe string:

```go
<p>
    Posted {{ .Date.Format "January 2, 2006" }}
    <span class="tags">
        in 
        {{ range .Params.tags }}
        <a class="tag" href="/tags/{{ . | urlize }}">{{ . }}</a>
        {{ end }}
    </span>
</p>
```

When you click on one of these tags, you are taken to a page that lists the pages associated with that tag. To create a custom layout, create the `themes/theme-name/layouts/tags.html` file and add custom content.

### Custom URLs

A _permalink_ is the full URL for any content page on your site. You can customize the format of the URL in `hugo.toml`. Here, we create a custom URL format for each blog post so they include the year and month:

```toml
[permalinks]
  posts = "posts/:year/:month/:slug"
```

Now, you have to add the `year` and `month` as front matter to the page. This example adds it to the archetype using Hugo functions:

```toml
+++
title = "{{ replace .Name "-" " " | title }}"
...
year = "{{ dateFormat "2006" .Date }}"
month = "{{ dateFormat "2006/01" .Date }}"
+++
```

Now, when you generate a new post, the year and month are automatically populated in the specified format. Any new posts are found at `site.com/posts/yyyy/MM/post-name/`. For example, `http://localhost:39939/posts/2025/07/second-post/`.

### Pagination

[Hugo pagination docs](https://gohugo.io/templates/pagination/)

Added this in `themes/docsite/layouts/posts/section.html`, but it doesn't seem to work each time:

```go
{{ define "main" }}
    <h2>{{ .Title }}</h2>

    {{ range (.Paginator 1).Pages }}
        {{ partial "post_summary.html" . }}
    {{ end }}

    {{ template "_internal/pagination.html" . }}
{{ end }}
```

### Displaying related content

You can link pages by adding the same `keywords` or `tags` to the front matter, then displaying the related content at the bottom of the page in a section called "Related Pages". For example, if post-1.md and post-2.md both have `keyword = ["posts"]` in the front matter, they will display in the related section on each page.

Here is the template markup:

```html
<section class="related">
    {{ $related := .Site.RegularPages.Related . | first 5 }}
    {{ with $related }}
        <h3>Related pages</h3>
        <ul>
            {{ range . }}
            <li><a href="{{ .RelPermalink }}">{{ .Title }}</a></li>
            {{ end }}
        </ul>
    {{ end }}
</section>
```
## Ch 6 Adding search to your site

Sites with database-driven searches have built-in search capabilities. The site takes the search terms and builds a query that is sent to the database that contains the content. The db returns the matching results, and the site displays it to the user.

Hugo doesn't use a database, so you need a solution that can index your site and search it:
- Algolia indexes your content and lets you search it
- ElasticSearch indexes your content
- You can generate your own search index and use client-side JS to perform the search

To generate your own search engine, you need Hugo to generate a list of your site content in JSON format.

### Creating the document collection

Lunr search requires these files:
- `theme/docsite/layouts/search.json`: Search index. Create this with Hugo templates.
- `content/search.md`: Search page. This has no content, only the output formats. The `search.json` file outputs the JSON to this page.
- `theme/docsite/layouts/search.html`: The search interface.
- `theme/docsite/static/js/search.js`: The JS search logic.



Your content list needs the page title and some text to search.


#### search.json

Create a search.json layout:

```json
{
    "results": [
        {{- range $index, $page := .Site.RegularPages }}
        {{- if $index -}} , {{- end }}
        {
            "href": {{ .Permalink | jsonify }},
            "title": {{ .Title | jsonify }},
            "body": {{ .Content | plainify | jsonify }}
        }
        {{- end }}
    ]
}
```

#### search.md

Create search content page. It won't have content--you need it to specify the output formats:

```toml
+++
...
outputs = ["HTML", "JSON"]
layout = 'search'
+++
```
This is enough to generate the search page at `/search/index.json`.

#### search.html

Next, create the search page where the user enters the search terms:

```go
{{ define "main" }}

    <h2>{{ .Title }}</h2>

    <input type="search" id="searchField">
    <button id="searchButton">Search</button>
    <input type="checkbox" id="allwords">
    <label for="allwords">Require all words</label>
    
    <div id="output">
        <p>Waiting for search input</p>
    </div>

    // lunr search library CDN
    <script src="//unpkg.com/lunr@2.3.6/lunr.js"></script>
    <script src="{{ "js/search.js" | relURL }}"></script>

{{ end }}
```

#### search.js

Fetch the JSON file and populate the search index:

```js
'use strict';

window.SearchApp = {
    searchField: document.querySelector("#searchField"),
    searchButton: document.querySelector("#searchButton"),
    allwords: document.querySelector('#allwords'),
    output: document.querySelector("#output"),
    searchData: {},
    searchIndex: {}
};

fetch('/search/index.json')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
    })
    .then(data => {
        SearchApp.searchData = data;
        SearchApp.searchIndex = lunr(function () {
            this.pipeline.remove(lunr.stemmer);
            this.searchPipeline.remove(lunr.stemmer);
            this.ref('href');
            this.field('title');
            this.field('body');
            data.results.forEach(e => {
                this.add(e);
            });
        });
    })
    .catch(error => {
        console.error('Error loading search index:', error);
    });
```

Next, build the search function and add an event listener to the search button on the search page:

```js
const search = () => {
    let searchText = SearchApp.searchField.value;
    if (searchText === '') return;          // return if empty

    searchText = searchText
        .split(' ')
        .map(word => `${word}*`)
        .join(' ');

    if (SearchApp.allwords.checked) {
        searchText = searchText
            .split(' ')
            .map(word => `+${word}`)
            .join(' ');
    }
    
    let resultList = SearchApp.searchIndex.search(searchText);

    let list = [];
    let results = resultList.map(entry => {
        SearchApp.searchData.results.filter(d => {
            if (entry.ref == d.href) {
                list.push(d);
            }
        });
    });
    display(list);
};

const display = (list) => {
    SearchApp.output.innerText = '';
    if (list.length > 0) {
        const ul = document.createElement('ul');
        list.forEach(el => {
            const li = document.createElement('li');
            const a = document.createElement('a');
            a.href = el.href;
            a.text = el.title;
            li.appendChild(a);
            ul.appendChild(li);
        });
        SearchApp.output.appendChild(ul);
    } else {
        SearchApp.output.innerHTML = "Nothing found";
    }
};

SearchApp.searchButton.addEventListener('click', search);
```

## Ch 7 Assets and pipes

The extended version of Hugo has pipes that let you transform your assets from within Hugo. For example:
- Minify JS and CSS makes your files transfer faster, which improves speed
- Fingerprint to each file, so when you deploy a modified version, the browser invalidates the cached version and fetches the new one.
- Image processing, including resizing images
- Hugo doesn't support advanced features like JS modules, so it integrates with Webpack.

### Directory structure

Pipelines require that you move your assets from the `static/` directory to `assets/`:

```bash
mkdir themes/docsite/assets
mv themes/docsite/static/css themes/docsite/assets
```

### Stylesheets

We're going to migrate to Sass, and minify and fingerprint the CSS assets:
- Sass lets has extended features like functions and lets you break your stylesheets into multiple files
- Minifying removes whitespace, non-printable characters, and comments so your files transfer faster.
- Fingerprinting creates a unique filename for your CSS files each time you modify it. When you serve content files, the browser caches it. A unique filename helps the browser notice that the file changed.

To use Sass, change the name of your CSS file:

```bash
mv themes/docsite/assets/css/style.css themes/docsite/assets/css/style.scss
```
With Sass, you can use partial files. For example, extract all navbar settings to `themes/docsite/assets/css/_navbar.scss`, and then import them into the `.../style.scss` stylesheet with the `@import "navbar";` statement at the top of the file.


To process the CSS assets, we store the CSS content location in a variable. Here, that is `$css`. `resources.Get` takes one argument: the path to the stylesheet within the `themes/docsite/assets` directory. The full path to this stylesheet is `themes/docsite/assets/css/style.css`:

```html
<!-- themes/docsite/layouts/_partials/head.html -->
{{ $css := resources.Get "css/style.scss" | toCSS | minify | fingerprint }}
```
Storing your CSS in a variable lets you apply transformations to the file:
- `toCSS` transforms your `.scss` file to `.css`
- `minify` minifies the files for faster file transfers
- `fingerprint` adds a unique ID to your files for caching

After you create the variable, use it in the `href` attribute for the `<link>`. This templating creates a relative link to the CSS file when it builds the `public/css` files:

```html
{{ $css := resources.Get "css/style.css" }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

These lines create these files in `public/`:

```bash
project-root/
│
... 
├── public
...
│   ├── css
│   │   └── style.min.6df00e795ff8de8f8f9b48a3b95d2f845a69f643faab7bc711f49736090b8738.css
│   ...
├── resources
│   └── _gen
│       └── assets
│           └── css
│               ├── style.scss_77b10c8e87ff110a62c52933fe3f7f11.content
│               └── style.scss_77b10c8e87ff110a62c52933fe3f7f11.json
...

```


#### Managing images

Theme and site static directories are merged together. You don't have to specify static in the path

This image is in `static/images/hugo-logo-wide.svg`.

```html
<!-- themes/docsite/layouts/_partials/footer.html -->
<p>Powered by <a href="https://gohugo.io">
        <img src="{{ "images/hugo-logo-wide.svg" | relURL }}" alt="image" width="128" height="38">
    </a>
</p>
```

You can include this in your markdown files like this:

```md
[This is an image](/hugo-logo-wide.svg)
```

### Page bundles

A page bundle is a content directory that includes all resources for a page. To create a page bundle, include the page bundle directory in the command. Here, `my-vacation` is the page bundle directory:

```bash
hugo new posts/my-vacation/index.md
project-root
│
...
├── content
│   ├── posts
│   │   ├── my-vacation                 # page bundle
│   │   │   ├── images
│   │   │   │   ├── badlands.jpg
│   │   │   │   ├── bison.jpg
│   │   │   │   └── rushmore.jpg
│   │   │   └── index.md
...
```

### Shortcodes

A shortcode is a template that outputs HTML. Store them in the `layouts/shortcodes/` directory:

```bash
mkdir themes/docsite/layouts/shortcodes
```

Here is a shortcode that shows how you can access resources in a page bundle:
1. `(.Get 0)` retrieves the first argument passed to the shortcode. Here, its a path.
   `$.Page.Resources.GetMatch` finds the page resource that matches the first argument path.
   `$.` calls the page context. Inside a shortcode, the context is the shortcode itself.
2. `$image.Resize "1024x"` resizes the image to 1024 pixels with the [Resize](https://gohugo.io/methods/resource/resize/) method. This uses `<width>x<height>` format. Specify width OR height so the image scales proportionately.
3. `.Get 1` retrieves the second argument to the shortcode. Here, its the figcaption text.

```html
(1) {{ $image := $.Page.Resources.GetMatch (.Get 0) }}
(2) {{ $smallImage := $image.Resize "1024x" }}

    <figure class="post-figure">
        <a href="{{ $image.RelPermalink }}">
            {{ with $smallImage }}
            <img 
                src="{{ .RelPermalink }}" 
                alt="{{ $.Get 1 }}"
                width="{{ .Width }}"
                height="{{ .Height }}">
            {{ end }}
        </a>
(3)     <figcaption>{{ .Get 1 }}</figcaption>
    </figure>
```

Here is how you use it in the content file:

```md
\{\{< postimage "images/rushmore.jpg" "Mount Rushmore" >}}
\{\{< postimage "images/badlands.jpg" "The Badlands" >}}
\{\{< postimage "images/bison.jpg" "Bison at Custer National Park" >}}
```

### Image processing

Processing images with Hugo can dramatically increase the build time. To speed things up, Hugo caches the converted images and saves them in the `resources/_gen/images/<path>` directory so it doesn't have to build them each time. 

If you change these image files, Hugo doesn't automatically clean the `resources/` directory, so you have to explicitly clear the cache on build:

```bash
hugo --gc
```

### Javascript

Put all your JS libraries in minified, fingerprinted file so you don't have to make network requests.

To use your JS files in a pipeline, move them from the `static/` directory to `assets/`:

```bash
mv themes/docsite/static/js/ themes/docsite/assets/
```

Make sure you have the full, unminified JS libraries. Here, we download the Lunr search library ino the `assets/` directory:

```bash
curl https://unpkg.com/lunr@2.3.6/lunr.js > themes/docsite/assets/js/lunr.js
```

Create your asset pipes where you import the library with script tags, in `themes/docsite/layouts/search.html`:
1. Get the `lunr.js` file resource object and store it in `$lunr`.
2. Get the `search.js` file resource object and store it in `$search`.
3. Combine the `$lunr` and `$search` resource objects into a slice named `$libs`.
4. `resources.Concat "js/app.js"` concatenates all files in `$libs` into a single file named `app.js`, then minifies and fingerprints the file.

```html
<!-- themes/docsite/layouts/search.html -->
...
(1) {{ $lunr := resources.Get "js/lunr.js" }}
(2) {{ $search := resources.Get "js/search.js" }}
 
(3) {{ $libs := slice $lunr $search }}
(4) {{ $js := $libs | resources.Concat "js/app.js" | minify | fingerprint }}
...
```
Here is the file after you build the project:

```bash
public/
│
...
├── js
│   └── app.min.4e058...20eaf.js
...
```

### Webpack

Hugo can minify, concatenate, and fingerprint files, but it can't do more complicated tasks like handle JS modules and transpilation.

First, create a `package.json` file. You can do this manually or with `npm init -y`:

```json
{
  "name": "docsite",
  "version": "1.0.0",
  "description": "Docsite theme",
  "private": true,                          // don't publish code to package repo
  "scripts": {
    "build": "hugo --cleanDestinationDir",
    "hugo-server": "hugo server --disableFastRender"    // nothing gets cached - hugo rerenders all pages on every change so its good if you're editing templates, styles, config files because all pages are updated
  },
  "author": ""
}
```

Build the site with the `build` command:

```bash
npm run build
```
To add webpack, you have to install it as a development dependency:

```bash
npm install --save-dev webpack webpack-cli
```

Install packages for the JS libraries that you want to include in the site. This package replaces any JS files that you have in `themes/docsite/assets/js`. These are production dependencies, so use `--save` instead of `--save-dev`:

```bash
npm install --save lunr
```
Webpack reads `webpack.config.js` for instructions on how to build your project. We are going to create a config file that looks for a file named `index.js`, resolves its dependencies (e.g. imports libraries), then outputs an asset file named `app.js`.

Create this file in the root of your site:

```js
(1) const path = require('path');

(2) module.exports = {
(3)     entry: './themes/docsite/assets/js/index.js',
(4)     output: {
(5)         filename: 'app.js',
(6)         path: path.resolve(__dirname, 'themes', 'docsite', 'assets', 'js')
        }
    };
```
1. Import the Node.js `path` module so you can join filesystem paths
2. Exports an object that Webpack uses as its configuration
3. Defines the entry point for your app. When building a dependency graph, Webpack begins with this file.
4. File name for the final bundled JS file
5. Path where the final bundles `app.js` file is written. `__dirname` is a special variable in Node.js that resolves to the current directory.

Because this is still in the `../assets/` folder, you can still use Hugo's minifier and fingerprinter. If you want to use Webpack for these pipeline stages, you should place the output in the `themes/../static/` directory and configure Webpack accordingly:

```js
...
module.exports = {
...
        path: path.resolve(__dirname, 'themes', 'docsite', 'static', 'js')
    }
};
```

Place any JS code you have in the `/themes/docsite/assets/js/index.js` file. Here, we had a file named `search.js` that we are converting into the entrypoint:

```bash
mv themes/docsite/assets/js/search.js themes/docsite/assets/js/index.js
```

Without webpack, we downloaded the JS library files and linked them in the `search.html` layout file. Because we downloaded the external libraries with Webpack, we can import them in our `index.js` instead:

```js
'use strict';

import lunr from 'lunr';
...
```

Webpack is downloading our JS dependencies, so we no longer need to concatenate the JS files as resource objects with `resources.Get` in `themes/docsite/layouts/search.html`. Instead, we retrieve the `app.js` resource that Webpack outputs. Because it is in the `assets/` directory, we can still use Hugo's `minify` and `fingerprint` functions:

```html
...
{{ $js := resources.Get "js/app.js" | minify | fingerprint }}
<script src="{{ $js.RelPermalink }}"></script>
```

In `package.json`, create a script that runs Webpack to build the `app.js` file before you run the Hugo server. This is necessary because Hugo needs to find this file in the `assets/` directory to build.

Webpack is installed as a project dependency, not as a global package, so you have to run Webpack directly from the binary in `node_modules/`. However, it is a project dependency, so you can run it with `package.json` without specifying the binary path in `node_modules`:

```js
{
  ...
  "scripts": {
    ...
    "webpack": "webpack"
  },
  ...
}
```

Webpack has a development server that watches for changes with the `webpack --watch` command, but that requires that you open two terminal servers to run Webpack and Hugo in parallel. Instead, you can install the `npm-run-all` command to run multiple tasks simultaneously:

```bash
npm install --save-dev npm-run-all
```

Next, add two scripts to `package.json`:

```js
{
  ...
  "scripts": {
    ...
    "webpack-watch": "webpack --watch",
    "dev": "npm-run-all webpack --parallel webpack-watch hugo-server"
  },
  ...
}
```

The `dev` script references the `webpack-watch` command that runs the Webpack dev server.

Finally, create a build script that runs Webpack and then Hugo. Rename the existing `build` script to `hugo-build`, and reference it in the new `build` command:

```js
{
  ...
  "scripts": {s
    "build": "npm-run-all webpack hugo-build",
    "hugo-build": "hugo --cleanDestinationDir",
    ...
  },
  ...
}
```

## Ch 8 deploying











## Commands

```bash

hugo new theme theme-name                   # create new theme
theme = "theme-name"                        # add new theme to config.toml
hugo new page-name.md                       # create new page in /content with default archetype
hugo new <archetype-name>/page-name.md      # create new page in /<archetyp-name> with /archetype file
hugo --cleanDestinationDir                  # delete /public and regen /public
hugo --cleanDestinationDir --minify         # minify /public contents
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
hugo --gc
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

### Webpack and npm

Webpack manages and builds frontend applications. Its powered by Node.js, so you have access to npm for package management and automation.

To use Webpack with Hugo, you have to do the following:
1. Install the dependencies
2. Create a `package.json` file with scripts that run Webpack and hugo
3. Create a `webpack.config.js` file that tells Webpack where to find your source JS files and build the output.
4. Add a path to your Webpack JS output file in your HTML.
