+++
title = 'Creating a Site'
date = '2025-07-13T09:25:20-04:00'
weight = 30
draft = false
+++



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