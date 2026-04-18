+++
title = 'Concepts'
date = '2025-07-12T09:59:22-04:00'
weight = 10
draft = false
summary = 'This is the page summary lorem ipsum'
+++


## Big ideas

> The layout structure changed with Hugo 1.46. Here is a breakdown: https://gohugo.io/templates/new-templatesystem-overview/. Notable changes include the following:
> - `layouts/_default/` directory is removed, and all layouts files go in the `layouts/`.
> - `layouts/index.html` is now `layouts/home.html`

- An archetype is a content template for a markdown page. A layout page is an HTML page that is generated from a markdown page. You need to associate a layout page with markdown pages generated from an archetype. For example, `layouts/<file>.html` is used for all pages.
  
  The home page is different--it generates `layouts/index.html` with content in `content/_index.md`.
- `layouts/baseof.html` is the skeleton for all layout files--the "base of" all files. It contains partials that define the head, header, main, etc.
- The default context for a layout page is the Page context (`.`).
- Partial functions take a filename and a context.


