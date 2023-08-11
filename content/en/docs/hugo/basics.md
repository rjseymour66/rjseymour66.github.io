---
title: "Basics"
linkTitle: "Basics"
weight: 30
description: >
  Gettings started
---

Hugo is a static site generator that takes Markdown files and HTML templates, then compiles it into HTML that you can serve on the internet.

## Creating a new site

Create a new site with the following command:

```bash
$ hugo new site <site-name> --format yaml
```

### Directory structure

After you create the site but before you add a theme, your website has the following directory structure:

```bash
root
├── archetypes
│   └── default.md
├── assets
├── config.yaml
├── content
├── data
├── layouts
├── public
├── resources
├── static
└── themes
```
These folders store the following content:
- `archetypes`: Templates for the content files.
- `assets`: Unprocessed images and JS/CSS files that are consumed globally for the website. Files in this directory can be processed during compilation. You process assets with Hugo Pipes.
- `config`: Website configuration, including any parameters that you pass to Hugo or the theme to render content. You can store configuration files for different environments, such as development, staging, and production.
- `content`: Content that you traditionally store in a database.
- `data`: Structured content that is made available to the website as global variables.
- `layouts`: Overrides parts of the theme with custom pages.
- `public`: Contains compiled output.
- `resources`: Optimized JS, CSS, and image files are cached here to make 
- `static`: Static content like fonts or PDF files. Hugo copies this content to the output directory, without processing.
- `themes`: Code that makes the content in the content folder presentable.

## Add a theme

Add the theme in the `themes/` directory, then add the theme in the `config.yaml` file: 

```bash
...
theme: <theme-name>
```

## Configuration

The `config.yaml` file has two sections:
1. Top-level configuration that is common across all themes.
2. Theme-specific parameter section that fill up menus, footers, copyright notices, and other information in the website.

## Add content

Add content in the `content/` directory. The directory structure defines the URL. For example, the `test.md` file is served at `https://localhost:1313/test`.