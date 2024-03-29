---
title: "Templates"
linkTitle: "Templates"
weight: 80
description: >
  How to add, work with, and process Templates.
---

## Need to detail each of these hugo objects, like site.RegularPages

Hugo compiles the template files in `layouts/` with the mardown content files to create HTML pages to serve to the browser.

## Template performance

Measure the impact of a template on the overall build with the `--templateMetrics` build flag:

```shell
$ hugo --templateMetrics
```


## Go templating language

Content outside of the double braces is treated as a raw string that you can pass into the final HTML. Information in the `content/` folder is available with variables, including the following top-level variables:
- `$`: The top-level context of the template. If we are using a Page template, we can access variables with the following declarations:
  - `$.Title` or `$.Page.Title`
  - `$.Description` or `$.Page.Description`
  
  Within a shortcode, the `$` operator is shortcode-scoped.
- `site`: Provides access to data for the whole website, including configuration data. This variable is available globally. Other examples include the following:
  - `site.Pages`: Access other pages in the site.
  - `site.Taxonomies`: Taxonomy data
  - `site.Params`: The `Params` object holds all the user-defined metadata in the front matter.

    Variables in `Params` are not case-sensitive. If the frontmatter key is `subtitle`, you can access it as `{{$.Params.Subtitle}}`
- `hugo`: Provides access to the Hugo compiler. For example:
  - `hugo.IsProduction`
  - `hugo.Version`

### Assets

The `assets/` folder is the global folder for the content accessible through Go templates.

### Existence checks

You can use an `if` statement to verify whether the value exists for a page. This lets you reuse templates across pages that might not have the same metadata or frontmatter, without fear of rendering empty strings in the HTML output.

The following examples uses `if` statements to render the page description and title when they exist. Because the title is rendered more than once, a variable is used in the existence check and then that variable is used to render the title throughout the page. The check uses the `site.Title` in the configuration as a default if there is not a local `Page.Title`:

```html
  {{if $.Description}}
  <meta name="description" content={{$.Description}}>
  {{end}}
  <link rel="stylesheet" href="./index.css">
  {{$title := $.Title}}
  {{if not $title}}
  {{$title = site.Title}}
  {{end}}

  {{if $title}}<title>{{$title}}</title>{{end}}
```
An equivalent variable assigment uses Hugo's `default` function:

```go
{{$title := default site.Title $.Title}}
```
In the previous example, the first argument to the `default` function is the default value, and the second is the value we check for existence.

`default` checks for the existence of a value, but an `if` statement checks whether the value is true or not (empty strings or 0 values do not pass the truthfulness test).

To conditionally render a value based on its truthfulness, use the `isset` function:

```go
{{if isset $.Params "subtitle"}}<h2>{{$.Params.subtitle}}</h2>{{end}}
```
The preceding example, we check if `$.Params` returns a value other than false (its true if it exists) and then conditionally render it.

### Param function pattern

The pattern of accessing a page variable and falling back to the site variable is so common that Hugo provides the built-in `$.Param` function to simplify this check. The following example provides the page subtitle and falls back to the site subtitle if the page value is not present:

```go
{{$.Param "subtitle"}}

// which is equivalent to
{{.Param "subtitle"}}
```

### Context switching with

To cut down on verbosity, you can change the current context from the Page to the codeblock with the `with` keyword. `with` does an existence check and sets the value of the variable to the `.` (dot) context variable.

The following two statements are equivalent:

```go
// Ex 1
{{if $title}}<h1>{{$title}}</h1>{{end}}

// Ex 2
{{with $title}}<title>{{.}}</title>{{end}}
```

`with` is helpful if the context variable is very verbose, like `site.Params.paramName[0].value`.


## Content

Add HTML content to a template with the `{{.Content}}` variable. This is available on the root context, but you do not need to include the `$`. The `$` is required only if you need to access the root context from within a `with` block.

### Content processing

- `humanize`: By default, Hugo templates use raw strings. This returns the humanized version of the argument in sentence-case.
  
  https://gohugo.io/functions/humanize/
- `markdownify`: Applies markdown processing.
  
  https://gohugo.io/functions/markdownify/

### Piping content

If you need to perform more than one processing step on your content, connect the steps with pipes:

```go
{{with $.Param "subtitle"}}<h2>{{. | humanize | markdownify}}</h2>{{end}}
```

### Looping through content

Loop through content with the `with` and `range` keywords. For example, you might want to create a section that lists all areas of the webpage. You can access the main areas in the main menu. Each entry looks like this:


```yaml
main:
  - identifier: about
    name: About
    url: /about
    weight: 100
  - identifier: contact
    name: Contact
    url: /contact
    weight: 200
```

The following code creates a website section using the main menu parameters:

```html
  {{/* Retrieve the values that you want to iterate through */}}
  {{ with site.Menus.main }}

  <section id="menu">
    <h1>Website sections</h1>
    <h2>This website has these major areas</h2>
    <ul>
      {{/* Use range to iterate through the current context, which is 
      (.) because the block uses with. */}}
      {{ range .}}
      <li>
        <a href="{{.URL}}">
          <i class="icon-{{.Identifier}}"></i>
          {{.Name | humanize}}
        </a>
        {{with .Post}}<p>{{.}}</p>{{end}}
      </li>
      {{ else }}
      {{/* Log for the website developer */}}
      {{end}}
    </ul>
  </section>
  {{end}}
```
First, you have to retrieve the values that you want to render. In this case, you use the `site.Menus.main` object. Hugo turns each of the menu values into subproperties of the object so you can access them.

Because this block uses the `with` keyword, the current context is `.`. So you can access any of the subproperties using the `.` and a capitalized name of the subproperty, such as `.Identifier`.

### Looping through specific sections

You can retrieve pages within a subdirectory of the `content/` directory with the following snippet:

```html
  {{with (where site.RegularPages ".Section" "in" site.Params.mainSections)}}
  <section id="blog">
    <h1>From our blog</h1>
    <ul class="posts">
      {{range first 3 .}}
      <li class="post">
        <a href="#{{.Permalink}}"></a>
        <h2>{{.Title}}</h2>
        <article>
          {{.Summary}}
        </article>
        <div>Read More</div>
      </li>
      {{end}}
    </ul>
  </section>
  {{end}}
```

The important parts of the snippet include the `with` statement that contains a nested `where` statement:

```go
{{with (where site.RegularPages ".Section" "in" site.Params.mainSections)}} 
```
This example does the following:
- Uses `with` to change the context. This also makes sure that you add nothing if the following `where` clause retreives no content.
- Uses the `where` function to tell Hugo where to retreive the files.
  
  It's similar to `WHERE` in sql: https://gohugo.io/functions/where/
- Retrieves a slice of regular pages with the `site.RegularPages` COLLECTION
- Filters the regular pages slice, looking for the `.Section` in the `site.Params.mainSections`. By default, it retrieves the section with the most content pages.

Another important feature of the snippet is the `range` section:

```go
{{range first 3 .}}
```
The `first` keyword slices an array to the first N elements. In this case, the elements are in the current context (`.`).


### Looping through frontmatter

Access the page frontmatter with the `.Param` property followed by the name of the frontmatter key:

```html
  {{with .Param "testimonials"}}

  <section id="testimonials">
    <h1>Custom endorsements</h1>
    <div>
      <ol>
        {{range .}}
        <li>
          <p>{{.content}}</p>
          <div>
            <h2>{{.author}}</h2>
            <h3>{{.from}}</h3>
          </div>
        </li>
        {{end}}
      </ol>
    </div>
  </section>
  {{end}}
```

## Parsing files

You can load data from files either from the `data/` directory, or the page bundle directory.


The following example uses the Resources API to fetch a file named `products.csv` and builds a table with its contents:

```html
{{with resources.GetMatch "products.csv"}}
<section id="products">
  <h1>Our Products</h1>
  {{with . | transform.Unmarshal (dict "delimiter" ",")}}
  <table>
    {{range $i, $value := .}}
    {{if eq $i 0}}<thead>{{end}}
      <tr>
        {{range $value}}
        <td>{{.}}</td>
        {{end}}
      </tr>
      {{if eq $i 0}}
    </thead>{{end}}
    {{end}}
  </table>
  {{end}}
  <a href="{{.Permalink}}" download>Download listing</a>
</section>
{{end}}
```

In the previous example:

1. We retrieve the `products.csv` file from the `assets/` directory with `resources.GetMatch`.
2. Parse the file with the `transform.Unmarshal` function. Pass this the CSV option `delimeter`, which returns a slice of slices. https://gohugo.io/functions/transform.unmarshal/#csv-options
3. Loop through the multidimensional slice with the `range` function as you would in Go code, with the key and value double assignment.
4. If the index is `0`, then create a table head element and continue processing.
5. Loop through the remaining values in the slice with the standard `range` loop.


## Content type

A _content type_ is a colleciton of templates that can render pages from markup according to a design. For example, a blog is a content type, an API endpoint is a topic type.

A _theme_ is a collection of content types that link to form a website. Each theme has a default type that all pages revert to if they are explicitly assigned a content type. For example, the `layouts/shortcodes/index.html` file is always the default type for the theme.

Hugo uses the default template that the theme provides unless you set the `type: _content type_` in the page's frontmatter. In addition, it uses the `content/` subdirectory name as the content type. For example, if you create a new file in the `content/endpoints/` directory, Hugo uses the `endpoints` content type if it exists. If it does not exist, it uses the default content type.

### Create a new content type

To create a content type, create a new folder in the `content/` directory, and name it using the name of the new content type. Then, place an `index.html` file in that directory.


### Base template

The _base template_ contains the common HTML parts of a web page (header, main, footer) that acts as a skeleton for other templates and partials to build on.

The base template should be generic so that it works well with all content types.

Create a `baseof.html` file in the `layouts/theme-name/` directory.

> The top-level context of a base template is the page.

### Blocks

A _block_ is content that you can override in specialized templates. Add blocks to the base template for sections that you expect child templates will override. You can provide a default value that is executed if a child template does not override the block. The following example shows how you define a block in the base template, and how you override it in a child template:

```html
<!-- baseof.html -->
{{block "blockname" blockArg}}
{{end}}

<!-- Child template -->
{{define "blockname"}}
<!-- use (.) to referene the blockArg, which is actually
     a context variable -->
{{end}}
```
The `define` keyword tells Hugo that the template defines blocks that are within the base template.

Blocks should have reasonable defaults, and any parameters should be optional.

### Layouts

A _layout_ is one of the different types of pages you can render using templates in Hugo. There are four main layouts:
- List layout: Renders index pages of sections within the website. This provides a list of all subsections and pages within each section.
- Single layout: Regular webpage with markdown content.
- Index layout: Renders the home page for the entire website and is present at the root.
- 404 layout: Base for all error pages within the site. Usually a 404 page is enough for static websites.

And there are two less important layouts:
- Taxonomy layout: Render the list of taxonomy tags.
- Terms layout: Render the list of pages associated with the taxonomy tags.

### Partials

_Inheritence_ is when a template uses the entire code of the base template and can modify certain parts (blocks).

_Composition_ is when you have a reusable piece of code that you can plug in wherever you want in the page.

A _partial_ is a reusable piece of code that you share among templates. They are equivalent to functions in standard programming in that they isolate computation and generate some output and return data. In general, they do the following:
1. Accept arguments
2. Process those arguments
3. Log data to the output HTML file
4. Return the results to the caller

Place partials directly under the `layouts/` folder---they are available across all content types in Hugo.

> You can override the partials used in a theme in your local project by creating a file with the same name in the `layouts/partials` folder. You can also use subdirectories as namespaces.



#### Args and contexts

Like shortcodes, a partial has access to no variables except the ones that you pass it. The `$` does not reference the global object---it references the top-level context that you pass to the partial. This promotes partial reuse and caching performance.

`$` and `.` are the same context when passed to a partial.

You must pass the context to blocks and partials.

Partials accept only one argument, so you have to use a dictionary to pass multiple values.

> **dict type**
> 
> The `dict` type creates a map from a list of k/v pairs. For example:
> ```go
> {{ partial "menu.html" (dict "Menu" site.Menus.main "Button" > true) }}
> ```
> 
> In the previous example, the `dict` passes the following:
> - `"Menu"` is the key, so you can reference it in the partial.
> - `site.Menus.footer` is the value that is accessible from the > `"Menu"` key. So, you can iterate over the `"Menu"` key in a > partial to access the footer elements for the site.
> - `"Button"` is another key set to `true`. This allows you to > conditionally render an element if the `"Button"` key is present > and set to true.
> 
> Below is the partial:
> ```html
{{with $.Menu }}
<nav>
    {{if $.Button}}
    <button class="hamburger">&#9776;</button>
    {{end}}
    <ul>
        {{range $.Menu}}
        <li>
            <a href="{{.URL}}">{{.Name | humanize}}</a>
            {{if .HasChildren}}
            <ul>
                {{ range .Children }}
                <li><a href="{{.URL}}">{{.Name | humanize}}</a></li>
                {{end}}
            </ul>
            {{end}}
        </li>
        {{end}}
    </ul>
</nav>
{{end}}
> ```
> 
> The previous example, the partial requires an argument of type `.> Menu` to process. This allows you to pass different menus on the > `site.Menus.menuType` parameter.

#### Nesting submenus

Hugo lets you infinitely nest submenus. You can access them with the `.HasChildren` and `.Children` properties.

Each menu item has a Boolean property `.HasChildren` and a slice array that contains any children named `.Children`. You can use the `range` function to conditionally iterate over this slice to render any children in a menu:

```html
...
{{range $.Menu}}
<li>
    <a href="{{.URL}}">{{.Name | humanize}}</a>
    {{if .HasChildren}}
    <ul>
        {{ range .Children }}
        <li><a href="{{.URL}}">{{.Name | humanize}}</a></li>
        {{end}}
    </ul>
    {{end}}
</li>
{{end}}
```

The previous example uses nested `range` loops to render menu items, and then conditionally render any submenus if the menu item's `.HasChildren` property returns true in the existence check.

[Menu variables](https://gohugo.io/variables/menus/)
[Menu templates](https://gohugo.io/templates/menu-templates/)


### Caching partials

If a partial is used more than a once, you can cache it to increase performance with `partialCached` function:

```go
{{ partialCached "menu.html" (dict "Menu" site.Menus.footer) "footer" }}
```

The previous example renders the footer from the `site.Menus.footer` object, and uses `"footer"` to create a variant that renders only the footer links (`"footer"` is a boolean in this example). This lets you cache distinct versions of a partial without having to execute the template multiple times.

[partialCached](https://gohugo.io/functions/partialcached/)

[Blog about partialCached](https://www.regisphilibert.com/blog/2019/12/hugo-partial-series-part-1-caching-with-partialcached/)

### Caching resource information with partials

You can temporarily hold data with the `scratch` data structure, which functions as a mutable dictionary. This is helpful because Hugo does not let you modify a slice or dict.