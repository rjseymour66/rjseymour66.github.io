+++
title = 'Content Sections'
date = '2025-07-13T09:28:21-04:00'
weight = 50
draft = false
+++



Archetypes are content templates. You can create a new archetype in your site or theme, then generate a markdown page using the template with this command:

```bash
hugo new <archetype>/filename.md
```
Hugo creates an `<archetype>` directory if it doesn't exist, and then create the `filename.md` file using the archetype as a template. For example, if you want to create a blog post, you create an archetype titled `posts.md`, then run this command to create your first post:

```bash
hugo new posts/first-post.md
```
Hugo first looks for the `posts` archetype in your site, then in the theme, and then it falls back to the `default.md` archetypes.

## Create a content section

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

## Section layouts

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

### Custom section layouts

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

## Section page content

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
