---
title: "Templates"
weight: 150
description: >
  Creating and working with templates in Go.
---

Templates can write dynamic webpages, config files, emails, etc. First, you have to parse the template, then you execute it.

## Overview

A template is composed of one or more templates. For example, you have a base template that contains the HTML metadata, header, body tags, and footer, and other parts (nav bar, main content) are templates that are injected into the base template. To create the templates, you have to parse and execute them. To save resources, you can creat a template cache that parses the templates once--when the server starts--and loads them as needed from a cache (a map). 

After you set up the template cache, you need to create a render method that executes a given template within the cache.

## Parse a template

You can 


These are the general steps:

1. Parse the contents of a template file:
    ```go
    t, err = template.ParseFiles(templateFile)
    ```
2. Create a struct that contains the content to inject into the template. The following struct injects the title and body of a webpage:
    ```go
    c := content {
        Title: "This is the title",
        Body:  "This is the body text",
    }
    ```
3. Execute the template, and write the executed template into a writer, such as a buffer:
    ```go
    if err := t.Execute(&buffer, c); err != nil {
        return nil, error
    }
    ```
4. Return or use the buffer somehow.


## Template composition

You can compose templates to minimize how often you have to write boilerplate html. A common pattern is to create a base template that invokes other templates within its HTML.

To define a distinct template, use the `{{define `_`template-name`_`}}` and `{{end}}` actions:

```go
{{define "base"}}
<!DOCTYPE html>
<html lang="en">
<head>
    ...
    <title>{{template "title" . }} - Snippetbox</title>
</head>
<body>
    ...
    <main>
        {{template "main" .}}
    </main>
    ...
</body>
</html>
{{end}}
```
The `{{template `_`template-name`_`. }}` actions invoke other templates named _`template-name`_. The `.` represents any dynamic data that you want to pass to the template.

Define the other templates in the relevant pages. For example, if you want to define the content that will be in the homepage, create a file called `home.tmpl.html` and define the templates:

```go
{{define "title"}}Home{{end}}

{{define "main"}}
    <h2>Latest Snippets</h2>
    <p>There's nothing to see here yet!</p>
{{end}}
```

## Parse composed templates

To parse a template composed of more than one template, you must do the following:
1. Create a struct containing the template file names.
2. Parse the files into a template set with the `template.ParseFiles()` function.
3. Execute the template, passing the name of the parent template to the `ExecuteTemplate()` function.

For example, the following handler parses composed templates to serve a website homepage:

```go
func home(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path != "/" {
        http.NotFound(w, r)
        return
    }

    // Initialize a slice containing the paths to the two files. It's important
    // to note that the file containing our base template must be the *first*
    // file in the slice.
    files := []string{
        "./ui/html/base.tmpl",
        "./ui/html/pages/home.tmpl",
    }

    // Use the template.ParseFiles() function to read the files and store the
    // templates in a template set. Notice that we can pass the slice of file
    // paths as a variadic parameter?
    ts, err := template.ParseFiles(files...)
    if err != nil {
        log.Print(err.Error())
        http.Error(w, "Internal Server Error", 500)
        return
    }

    // Use the ExecuteTemplate() method to write the content of the "base" 
    // template as the response body.
    err = ts.ExecuteTemplate(w, "base", nil)
    if err != nil {
        log.Print(err.Error())
        http.Error(w, "Internal Server Error", 500)
    }
}
```

## Partials

A partial is a smaller template component that you might reuse in multiple pages. For example, a navigation bar.



## Dynamic data

As stated earlier, the dot (`.`) represents any dynamic data that you want to pass to the template.

You can pass dynamic data when you execute the template. For example, you can pass a struct to the template set:

```go
err = ts.ExecuteTemplate(w, "base", snippet)
if err != nil {
    app.serverError(w, err)
}
```
When the template set is executed, it will include `snippet` as its dynamic data. If `snippet` is a struct (in this example, it is a struct), you can render the value of an exported field in your template by postfixing the dot with the field name.

For example, if you pass a snippet object to a template set:

```go
type Snippet struct {
	ID      int
	Title   string
	Content string
	Created time.Time
	Expires time.Time
}
```

You can dynamically render these any exported fields in a template, as follows:

```html
{{define "title"}}Snippet #{{.ID}}{{end}}

{{define "main"}}
<div class="snippet">
    <div class="metadata">
        <strong>{{.Title}}</strong>
        <span>#{{.ID}}</span>
    </div>
    <pre><code>{{.Content}}</code></pre>
    <div class="metadata">
        <time>Created: {{.Created}}</time>
        <time>Expires: {{.Expires}}</time>
    </div>
</div>
{{end}}
```

### Method access

If the underlying data type of a rendered field has a method, you can invoke that method and render it. For example, `Snippet.Created` is of type `time.Time`, so you can access the `.Weekday` method:

```html
<span>{{.Snippet.Created.Weekday}}</span>
```

If you access a method that accepts arguments, then you pass them by listing them after the method name, with a space between each argument:

```html
<span>{{.Snippet.Created.AddDate 0 6 0}}</span>
```


### Multiple pieces of dynamic data

By default, you can only pass one piece of dynamic data when executing a template set. A common workaround is to wrap the dynamic data in a struct.

The struct is just a wrapper--do not worry about the logical grouping of the data. Include any data that you want to dynamically inject into the template during execution.

To wrap the dynamic data, you must create a `templates.go` file in your `/cmd/web` directory, and wrap the data:

```go
type templateData struct {
	CurrentYear int
	Snippet     *models.Snippet
	Snippets    []*models.Snippet
}
```

You use this dynamic data in your handlers.

Next, instantiate the wrapper struct, and pass it to the template set during execution:

```go
func (app *application) handlerView(w http.ResponseWriter, r *http.Request) {
    ...
    // Create an instance of a templateData struct holding the snippet data.
    data := &templateData{
        Snippet: snippet,
    }

    // Pass in the templateData struct when executing the template.
    err = ts.ExecuteTemplate(w, "base", data)
    if err != nil {
        app.serverError(w, err)
    }
}
```


Use dot notation to access the fields in the template:

```html
{{define "title"}}Snippet #{{.Snippet.ID}}{{end}}

{{define "main"}}
<div class="snippet">
    <div class="metadata">
        <strong>{{.Snippet.Title}}</strong>
        <span>#{{.Snippet.ID}}</span>
    </div>
    <pre><code>{{.Snippet.Content}}</code></pre>
    <div class="metadata">
        <time>Created: {{.Snippet.Created}}</time>
        <time>Expires: {{.Snippet.Expires}}</time>
    </div>
</div>
{{end}}
```

### Pipelining nested templates

When you invoke a template from within another template, you must pipeline the dot to the template you are invoking. To do so, include the dot at the end of each `template` or `block` action:

```html
<body>
    <header>
        <h1><a href="/">Snippebox</a></h1>
    </header>
    {{template "nav" .}}
    <main>
        {{template "main" .}}
    </main>
    <footer>Powered by <a href="https://golang.org/">Go</a></footer>
    <script src="/static/js/main.js"></script>
</body>
```
In the preceding example, the dynamic data is passed to the "nav" and "main" templates.

## Actions

_Actions_ are components that render data using conditional logic and the value of dot.

### Blocks

A block is a template that can include default content if the page invokes a template that is not in the executed template set.

### if
`{{if}}` renders content conditionally, with the `{{else}}` and `{{end}}` actions.

### with

`{{with}}` changes the value of dot. When dot holds a value, any dot within its scope is set to its value.

For example, the dot in the following template represents the templateData struct wrapper:

```go
type templateData struct {
	Snippet  *models.Snippet
	Snippets []*models.Snippet
}
```

```html
{{define "title"}}Snippet #{{.Snippet.ID}}{{end}}

{{define "main"}}
<div class="snippet">
    <div class="metadata">
        <strong>{{.Snippet.Title}}</strong>
        <span>#{{.Snippet.ID}}</span>
    </div>
    <pre><code>{{.Snippet.Content}}</code></pre>
    <div class="metadata">
        <time>Created: {{.Snippet.Created}}</time>
        <time>Expires: {{.Snippet.Expires}}</time>
    </div>
</div>
{{end}}
```

You can use `{{with}}` to pass the `.Snippet` value to any dot within its scope:

```html
{{define "title"}}Snippet #{{.Snippet.ID}}{{end}}

{{define "main"}}
    {{with .Snippet}}
    <div class="snippet">
        <div class="metadata">
            <strong>{{.Title}}</strong>
            <span>#{{.ID}}</span>
        </div>
        <pre><code>{{.Content}}</code></pre>
        <div class="metadata">
            <time>Created: {{.Created}}</time>
            <time>Expires: {{.Expires}}</time>
        </div>
    </div>
    {{end}}
{{end}}
```
If there is a value assigned to `templateData.Snippet`, then any dots between `{{with .Snippet}}` and `{{end}}` become `Snippet`, rather than `templateData`.

### range
`{{range}}` changes the value of dot. It loops over data passed to the template:

```html
{{range .Snippets}}
<tr>
    <td><a href="/snippet/view?id={{.ID}}">{{.Title}}</a></td>
    <td>{{.Created}}</td>
    <td>{{.ID}}</td>
</tr>
{{end}}
```

You can use `{{if}}`, `{{continue}}`, and `{{break}}` during range loops just as you would in any C-derived language.

## Functions

The following table describes the most common templating functions:

| Function | Description |
|:---------|:------------|
| {{eq .Foo .Bar}} | Yields true if .Foo is equal to .Bar |
| {{ne .Foo .Bar}} | Yields true if .Foo is not equal to .Bar |
| {{not .Foo}} | Yields the boolean negation of .Foo |
| {{or .Foo .Bar}} | Yields .Foo if .Foo is not empty; otherwise yields .Bar |
| {{index .Foo i}} | Yields the value of .Foo at index i. The underlying type of .Foo must be a map, slice or array, and i must be an integer value. |
| {{printf "%s-%s" .Foo .Bar}} | Yields a formatted string containing the .Foo and .Bar values. Works in the same way as fmt.Sprintf(). |
| {{len .Foo}} | Yields the length of .Foo as an integer. |
| {{$bar := len .Foo}} | Assign the length of .Foo to the template variable $bar |

> In Go templating, _yeild_ means _render_.

## Caching templates

You do not want to parse template files each time you render a web page. The correct strategy is to create a map (a cache) that stores a template set for each page you want to render and a helper function that executes the specified template set.

First, create the template cache in `templates.go`:

```go
func newTemplateCache() (map[string]*template.Template, error) {
    // Initialize a new map to act as the cache.
    cache := map[string]*template.Template{}

    // Use the filepath.Glob() function to get a slice of all filepaths that
    // match the pattern "./ui/html/pages/*.tmpl". This will essentially gives
    // us a slice of all the filepaths for our application 'page' templates
    // like: [ui/html/pages/home.tmpl ui/html/pages/view.tmpl]
    pages, err := filepath.Glob("./ui/html/pages/*.tmpl.html")
    if err != nil {
        return nil, err
    }

    // Loop through the page filepaths one-by-one.
    for _, page := range pages {
        // Extract the file name (like 'home.tmpl') from the full filepath
        // and assign it to the name variable.
        name := filepath.Base(page)

        // Parse the base template file into a template set.
        ts, err := template.ParseFiles("./ui/html/base.tmpl.html")
        if err != nil {
            return nil, err
        }

        // Call ParseGlob() *on this template set* to add any partials.
        ts, err = ts.ParseGlob("./ui/html/partials/*.tmpl.html")
        if err != nil {
            return nil, err
        }

        // Call ParseFiles() *on this template set* to add the  page template.
        ts, err = ts.ParseFiles(page)
        if err != nil {
            return nil, err
        }

        // Add the template set to the map, using the name of the page
        // (like 'home.tmpl') as the key.
        cache[name] = ts
    }

    // Return the map.
    return cache, nil
}
```
The `newTemplateCache()` function creates a template set for each page in the `/ui` directory with the following steps:
1. Creates a new map to cache the templates in memory
2. Searches the `/ui` directory for files with the project's template extension and stores them in a slice. In this example, the file extension is `.tmpl.`
3. Loop through the slice. For each template file, do the following:
   - Store the file name
   - Parse the base template
   - Find any partials, parse them and add them to the template set
   - Parse the current template file, adding it to the template set
   - Add the current template set to the cache

Next, write a helper function to render each template set in the cache. The helper should perform the following:
- Get the correct template set from the cache
- Create a bytes buffer to execute the template. This allows error checking.
- Write an HTTP header
- Write the buffer to the writer.

This helper simplifies handler functions:

```go
func (app *application) render(w http.ResponseWriter, status int, page string, data *templateData) {
    // Retrieve the appropriate template set from the cache based on the page
    // name (like 'home.tmpl'). If no entry exists in the cache with the
    // provided name, then create a new error and call the serverError() helper
    // method that we made earlier and return.
    ts, ok := app.templateCache[page]
    if !ok {
        err := fmt.Errorf("the template %s does not exist", page)
        app.serverError(w, err)
        return
    }

    // Initialize a new buffer.
    buf := new(bytes.Buffer)

    // Write the template to the buffer, instead of straight to the
    // http.ResponseWriter. If there's an error, call our serverError() helper
    // and then return.
    err := ts.ExecuteTemplate(buf, "base", data)
    if err != nil {
        app.serverError(w, err)
        return
    }

    // If the template is written to the buffer without any errors, we are safe
    // to go ahead and write the HTTP status code to http.ResponseWriter.
    w.WriteHeader(status)

    // Write the contents of the buffer to the http.ResponseWriter. Note: this
    // is another time where we pass our http.ResponseWriter to a function that
    // takes an io.Writer.
    buf.WriteTo(w)
}
```

Finally, use the helper to the handler. Add the helper after you perform any error checking:

```go
func handlerName(w, r) {
    // error checking
    // retrieve data from db

    app.render(w, http.StatusOK, "home.tmpl", &templateData{
            Datas: data,
        })
}
```
## Custom template functions

You can create your own functions that you can pass to templates in the same way that you use `{{eq}}` or `{{len}}`. The function can return only one value and an optional `error`.

To create a custom template function, follow these steps:
1. Write your function.
2. Store the function in a [FuncMap](https://pkg.go.dev/text/template#FuncMap).
3. Register the `FuncMap` with the template set with the following steps:
   1. Create an empty template set with `New()`
   2. Register the `FuncMap` with `Funcs()`.
   3. Parse the template set.

```go
// Create a humanDate function which returns a nicely formatted string
// representation of a time.Time object.
func humanDate(t time.Time) string {
    return t.Format("02 Jan 2006 at 15:04")
}

// Initialize a template.FuncMap object and store it in a global variable. This is
// essentially a string-keyed map which acts as a lookup between the names of our
// custom template functions and the functions themselves.
var functions = template.FuncMap{
    "humanDate": humanDate,
}

func newTemplateCache() (map[string]*template.Template, error) {
    ...

    for _, page := range pages {
        name := filepath.Base(page)

        // The template.FuncMap must be registered with the template set before you
        // call the ParseFiles() method. This means we have to use template.New() to
        // create an empty template set, use the Funcs() method to register the
        // template.FuncMap, and then parse the file as normal.
        ts, err := template.New(name).Funcs(functions).ParseFiles("./ui/html/base.tmpl")
        if err != nil {
            return nil, err
        }
        ...
    }

    return cache, nil
}
```

Use the custom functions just as you would use native template functions:

```html
<div class="metadata">
    <time>Created: {{humanDate .Created}}</time>
    <time>Expires: {{humanDate .Expires}}</time>
</div>
```
You can also use Unix pipes to chain functions:

```html
<div class="metadata">
    <time>Created: {{.Created | humanDate}}</time>
    <time>Expires: {{.Expires | humanDate | printf "Created: %s"}}</time>
</div>
```