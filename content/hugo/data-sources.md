+++
title = 'Data Sources'
date = '2025-07-13T09:29:40-04:00'
weight = 60
draft = false
+++



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