+++
title = 'Themes'
date = '2025-07-13T09:26:28-04:00'
weight = 40
draft = false
+++


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