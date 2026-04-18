+++
title = 'Common Features'
date = '2025-07-13T09:31:04-04:00'
weight = 70
draft = false
+++




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