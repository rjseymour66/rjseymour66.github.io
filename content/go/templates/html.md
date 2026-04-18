+++
title = 'HTML'
date = '2025-09-16T22:38:19-04:00'
weight = 10
draft = false
+++

Go has a templating engine that lets you return text and HTML in response to HTTP requests. The HTML library is built on top of the text engine.

The templating engine is context-aware, which means it understands HTML, CSS, JS, and URIs and escapes data based on where it appears in the HTML output. In short, developers are trusted and any user data that is injected with variables is untrusted and is escaped. This is a security measure to prevent attacks such as cross-site scripting (XSS) from untrusted data.

For example, if a user input the following:

```html
<script>alert('malicious')</script>
```

The HTML template package escapes text to output this:

```html
&lt;script&gt;alert(&#39;malicious&#39;)&lt;/script&gt;
```

## Data context

This table gives a brief overview of Go's "dot" syntax:

| Syntax                  | Meaning                                |
| ----------------------- | -------------------------------------- |
| `{{ . }}`               | Print the current value                |
| `{{ .Field }}`          | Access a field of the current value    |
| `{{ range .Items }}`    | Iterate items and set `.` to each item |
| `{{ with .SubValue }}`  | Temporarily set `.` to `.SubValue`     |
| `{{if}}` and `{{else}}` | Perform conditional checks             |

The dot represents the current context in that scope. This is best demonstrated with the `range` action, which uses the top-level context and the context nested within each iteration of the loop. The following two `range` expressions evaluate `[]people` equivalently:
1. `range` over `people`.
2. `item` in `people` slice.

```go
{{ range . }} 						// 1
<li>{{ . }}</li> 					// 2
{{ end }}

for _, item := range people {		// 1
	item							// 2
}
```


## Getting started

This example demonstrates the basics of HTML templating. It renders the title and content in this simple web page. Here is the returned HTML located at `./html/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{.Title}}</title>
  </head>
  <body>
    <h1>{{.Title}}</h1>
    <p>{{.Content}}</p>
  </body>
</html>
```

Here is the code to replace `{{.Title}}` and `{{.Content}}` with the templating engine:
1. Create a struct that holds data you want to inject into the template.
2. A handler that loads the template data and executes the template.
3. The `Page` struct holds the data that you inject into the template.
4. This line does a few things:
   - `template.ParseFiles` takes a path to a template file. It loads the HTML file for processing.
   - `template.Must` is a helper function that you can wrap around `template.ParseFiles`. `template.ParseFiles` returns a `*Template` and an `error`. If it returns an `error`, then `Must` panics and stops execution. Otherwise, it returns the `*Template` type and continues processing. This code is equivalent to this:
   ```go
   t, err := template.ParseFiles("html/index.html")
   if err != nil {
       panic(err)
   }
   ```
5. `Execute` executes the template, which substitutes the `Page` values for the `{{.Title}}` and `{{.Content}}` values in the template and writes them to the `Writer`.
   

```go
type Page struct {                                                  // 1
	Title, Content string
}

func displayPage(w http.ResponseWriter, r *http.Request) {          // 2
	p := &Page{                                                     // 3
		Title:   ".Title example",
		Content: "This is the .Content block.",
	}

	t := template.Must(template.ParseFiles("html/index.html"))      // 4
	t.Execute(w, p)                                                 // 5
}

func main() {
	http.HandleFunc("/", displayPage)
	http.ListenAndServe(":8080", nil)
}
```

Alternately, you could instantiate the `Page` object in `t.Execute`:

```go
func displayPage(w http.ResponseWriter, r *http.Request) {
	t := template.Must(template.ParseFiles("html/index.html"))
	t.Execute(w, Page{
		Title:   ".Title example",
		Content: "This is the .Content block.",
	})
}
```

## Template sets

If your app uses more than one template file, you can create a template set that stores multiple HTML templates. A template set is a group of templates that are parsed together and share a namespace. Each template contains a pointer to a parse tree---a structured representation of markup after it is parsed:

```go
type Template struct {
	*parse.Tree
}
```

When you parse a template with `ParseFiles` or `ParseGlob`, Go creates a tree structure for each template, where each node in the tree represents a part of the template. A node can be plain text, a loop, or an action, so the tree becomes a structured set of instructions to create your output.

A template set is a container that maps template names to the parse tree for the associated template.


Then, you can render an individual template from that set in a handler:
1. `template.New()` creates and returns a template set `t`. This is a container for all HTML templates that you want to prepare for rendering. You will not often (ever?) reference the template set name after you create it with `New()`.
2. `t.ParseGlob` reads every HTML file in the specified path pattern and parses them into the template set you created with `New()`. In Go, the `init()` function always runs before `main`.
3. Within the handler, you call `t.ExecuteTemplate` to look up a template file in your template set `t` and execute it with (inject) the `Page` object `p` as your data context.

```go
type comment struct {
	Username string
	Text     string
}

type Page struct {
	Title, Content string
	Comments       []comment
}

var t = template.New("templates")                                           // 1

func init() {
	_, err := t.ParseGlob("html/*.html")                                    // 2
	if err != nil {
		log.Fatal("Error loading templates:" + err.Error())
	}
}

func main() {
	http.HandleFunc("/comments", routeComments)
	if err := http.ListenAndServe(":8085", nil); err != nil {
		panic(err)
	}
}

func routeComments(w http.ResponseWriter, r *http.Request) {
	p := &Page{
		Title:   "An Example",
		Content: "Have fun stormin' da castle.",
		Comments: []comment{
			{Username: "Bill", Text: "Looks like a good example."},
			{Username: "Jill", Text: "I really enjoyed this article."},
			{Username: "Phil", Text: "I don’t like to read."},
		},
	}
	if err := t.ExecuteTemplate(w, "list.html", p); err != nil {            // 3
		log.Println(err)
		w.WriteHeader(http.StatusInternalServerError)
	}
}
```

## Template functions

Go templates provide helper functions to process your data into templates, but you might need additional functionality not available in the template library. You can create custom functions and use them in your templates with `FuncMap`.

After you create a function, create a function map with `template.FuncMap`. `.FuncMap` is a map of string names of functions to a custom function:

```go
var funcMap = template.FuncMap{
    // "reference": functionName
	"dateFormat": dateFormat,
}
```

You can make them available to your template with `.Funcs(funcMap)`. First you have to parse your template file, then add it to your template:

```go
func serveTemplate(w http.ResponseWriter, r *http.Request) {
	t := template.New("date")
	t.Funcs(funcMap)
    // ...
}
```

Here is a full example that demonstrates how you can create custom functions and add them to Go's template engine:
1. Create a template with a `string`. Notice how the `dateFormat` function is invoked.
2. Create a `FuncMap` with a custom `dateFormat` function.
3. `dateFormat` function implementation. This function takes a layout string and a `time` value. In the template string, the function is invoked in the following format:
   ```go
   {{ .Date | dateFormat "Jan 2, 2006" }}
   ```
   When you pipe data to a template function, the value on the left of the pipe becomes the last argument to the funciton on the right. This line translates to `dateFormat("Jan 2, 2006", .Date)`.
4. Create a new template set named "date".
5. Add `funcMap` to the template's function map.
6. Use `Parse` to parse a text string as a template body.
7. An anonymous struct named `data` with a `Date` field that contains the current time.
8. Execute the template. Write the template to the response `Writer`, and inject the `data` struct.

```go
var tpl = `<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Date example</title>
  </head>
  <body>
    <p>{{ .Date | dateFormat "Jan 2, 2006" }}</p>
  </body>
</html>`                                                        // 1

var funcMap = template.FuncMap{                                 // 2
	"dateFormat": dateFormat,
}

func dateFormat(layout string, d time.Time) string {            // 3
	return d.Format(layout)
}

func serveTemplate(w http.ResponseWriter, r *http.Request) {
	t := template.New("date")                                   // 4
	t.Funcs(funcMap)                                            // 5
	t.Parse(tpl)                                                // 6
	data := struct{ Date time.Time }{                           // 7            
		Date: time.Now(),
	}
	t.Execute(w, data)                                          // 8
}

func main() {
	http.HandleFunc("/", serveTemplate)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```

## Caching templates

The [preceding example](#template-functions) parses templates in the handler, which means that your application must parse the templates each time a request is made to that endpoint. This reduces performance.

Rather than parse the templates in a handler, parse them and store them in a package-level variable so all handlers or methods can use the templates:
1. Create a global template set. 
2. Within the `displayPage` handler, create a `Page` object, and inject it into the template set when you execute it.

```go
var t = template.Must(template.ParseFiles("html/index.html"))       // 1

type Page struct {
	Title, Content string
}

func displayPage(w http.ResponseWriter, r *http.Request) {
	p := &Page{
		Title:   "An Example",
		Content: "This is example content",
	}
	t.Execute(w, p)                                                 // 2
}
func main() {
	http.HandleFunc("/", displayPage)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```

## Error handling with a buffer

To prevent errors in your templates, you can write to a buffer first, handle any errors, then write the buffer to the `Writer`. This approach also facilitates testing---writing to a buffer lets you verify whether you are writing the correct values:
1. Create a global template set variable.
2. Use an `init` function to load and parse templates at startup. If there is an error, `template.Must` stops execution before `main` runs.
3. In the handler, create a buffer
4. Execute the template with the injected data and store its output in the buffer.
5. Handle any errors that might occur.
6. Write the contents of the buffer to the response `Writer`.


```go
var t *template.Template                                            // 1
func init() {                                                       // 2
	t = template.Must(template.ParseFiles("html/index.html"))
}

type Page struct {
	Title, Content string
}

func displayPage(w http.ResponseWriter, r *http.Request) {
	p := &Page{
		Title:   "An Example",
		Content: "This is example content",
	}
	var b bytes.Buffer                                              // 3
	err := t.Execute(&b, p)                                         // 4
	if err != nil {                                                 // 5
		fmt.Fprint(w, "An error occurred")
		return
	}
	b.WriteTo(w)                                                    // 6
}
func main() {
	http.HandleFunc("/", displayPage)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```

## Nested templates

Many sections of a web page are identical. You can nest templates within each other to compose a single large template file from two or more smaller template files.

For example, most web pages have an identical `<head>` element. You can create a `head` template and share it across all web pages, which reduces duplication.

### Sharing data

To add a template to another, use the `{{template <template> .}}` directive, which consists of three parts:
- `template`: Tells the template engine to include another template at this location.
- `<template> `: Name of the template to include.
- `.`: Dataset to pass to the template. A dot (`.`) passes the dataset from the parent template, also called the "current context".

To illustrate, here is a `<head>` template and an index file that injects the `<head>` template:
```html
<!-- head.html -->
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{ .Title }}</title>
</head>

<!-- index.html -->
  <!DOCTYPE html>
<html lang="en">
  {{template "head.html" .}}
  <body>
    <h1>{{ .Title }}</h1>
    <p>{{ .Content }}</p>
  </body>
</html>
```
In the `<head>` template, the `<title>` element contains `{{.Title}}`, which renders the `Title` data from the parent dataset. This is the same `Title` that is used in the `<h1>` element in `index.html` because we passed the parent dataset to the `<head>` template, which is the same dataset available to `index.html`.

### Serving the files

The code is similar to other examples on this page, except that the `init` function parses all template files that compose the web page and the handler specifies which template to render:
1. Parses both template files into the same template set. This makes `head.html` available to `index.html`.
2. The handler calls `ExecuteTemplate` to write the template and its data to the Writer. This function lets you specify which template to render. Other examples on this page use the `Execute` function, which executes the first file passed to `ParseFiles`.

```go
var t *template.Template

func init() {
	t = template.Must(template.ParseFiles("html/index.html", "html/head.html"))     // 1
}

type Page struct {
	Title, Content string
}

func displayPage(w http.ResponseWriter, r *http.Request) {
	p := &Page{
		Title:   "An Example",
		Content: "This is example content",
	}
	t.ExecuteTemplate(w, "index.html", p)                                           // 2
}
func main() {
	http.HandleFunc("/", displayPage)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```

## Inheritance

Template inheritance lets you compose multiple unique web pages from a single base template. A common pattern uses a base template defines the structure of the page by invoking additional templates and their data context. For example, you might create a base template that invokes a head template that renders a different `<title>` element for each page. In this pattern, it is said that the content templates "extend" the base template.

### Base templates

Here is a `base.html` template file. This uses a few key topics that are key to understanding how templates are composed:
1. `define` directive that starts the "base" template. All `define` directives must have an `{{end}}` tag.
2. Invokes the `title` template, which is defined in another file.
3. `block` directives define and immediately execute a template. Here, it defines a template named `styles` that adds CSS to the file.

```go
{{define "base" -}}                                 // 1
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{template "title" .}}</title>           // 2
    {{ block "styles" . -}}                         // 3
    <style>
      h1 {
        color: #400080;
      }
    </style>
    {{- end }}
  </head>
  <body>
    <h1>{{template "title" .}}</h1>
    {{template "content" .}}
    {{block "scripts" .}}{{end}}
  </body>
</html>
{{- end}}
```

{{< admonition "Trimming whitespace" tip >}}
By default, Go templates respect whitespace and newline characters created by the templating directives. To remove the whitespace and newlines, add dashes inside the double brackets before or after the whitespace:
- `{{- end}}`: removes before
- `{{define "base" -}}`: removes after
- `{{- end -}}`: removes before and after
{{< /admonition >}}

### Extending the base

To extend the `base` template, the extending file must defines all sections invoked from the `base` template. In this case, the extending templates must define a `title` and `content` section.

The `user.html` template file defines both the `title` and `content` templates that are pulled into the preceding `base` template:
1. `title` template definition and end.
2. `content` template definition.
3. Closing the `content` template with `{{end}}`.

```go
{{define "title"}}User: {{.Username}}{{end}}        // 1
{{define "content"}}                                // 2
<ul>
    <li>Username: {{.Username}}</li>
    <li>Name: {{.Name}}</li>
</ul>
{{end}}                                             // 3
```

### Parsing the templates


Here, we parse multiple files with the `base.html` template to store them in a map:
1. Create a map with `string` keys and `Template` template object values.
2. Use a `temp` variable to parse template objects in place. First, parse the `user.html` file with `base.html`.
3. Store the template object in the `t` map with `user.html` as the key.
4. Repeat this parsing process with `index.html`, and store it in the map with `index.html` as its key.
5. Execute each template with the `ExecuteTemplate` function. To call the correct template, use its key name in the map. For example, `displayPage` executes the template object located at `t["index.html"]` and injects `Page` data.

```go
var t map[string]*template.Template                     // 1

func init() {
	t = make(map[string]*template.Template)

	temp := template.Must(template.ParseFiles("html/base.html", "html/user.html"))  // 2
	t["user.html"] = temp                                                           // 3

	temp = template.Must(template.ParseFiles("html/base.html", "html/index.html"))  // 4
	t["index.html"] = temp
}

type Page struct {
	Title, Content string
}

type User struct {
	Username, Name string
}

func displayPage(w http.ResponseWriter, r *http.Request) {
	p := &Page{
		Title:   "An Example",
		Content: "This is example content",
	}
	t["index.html"].ExecuteTemplate(w, "base", p)               // 5
}

func displayUser(w http.ResponseWriter, r *http.Request) {
	u := &User{
		Username: "swordsmith",
		Name:     "Inigo Montoya",
	}
	t["user.html"].ExecuteTemplate(w, "base", u)
}

func main() {
	http.HandleFunc("/", displayPage)
	http.HandleFunc("/user", displayUser)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```


## Sending email

Many service notifications, registrations, etc are communicated through email. Because we don't need the `html/template` package's context-awareness, we can use Go's `text/template` package to send plaintext emails.

This example sends a simple mail transfer protocol (SMTP) message:
1. Struct that holds email components.
2. Struct that holds SMTP server credentials.
3. Template string for the email body.
4. Global template set.
5. Use `init` to create a new template set before `main` starts.
6. Parse the template string as a template body.
7. Create an `EmailMessage`.
8. Create a bytes buffer and write the formatted email message to the buffer.
9. Create an object that contains the email server authentication information.
10. `PlainAuth` creates a PLAIN authentication mechanism.
11. Send the email with `SendMail`.
    
```go
type EmailMessage struct {                          // 1
	From, Subject, Body string
	To                  []string
}

type EmailCredentials struct {                      // 2
	Username, Password, Server string
	Port                       int
}

                                                    // 3
const emailTemplate = `From: {{.From}}              
To: {{.To}}
Subject {{.Subject}}
{{.Body}}
`

var t *template.Template                            // 4

func init() { 
	t = template.New("email")                       // 5
	t.Parse(emailTemplate)                          // 6
}

func main() {
	message := &EmailMessage{                       // 7
		From:    "me@example.com",
		To:      []string{"you@example.com"},
		Subject: "A test",
		Body:    "Just a quick hello",
	}

	var body bytes.Buffer                           // 8
	t.Execute(&body, message)

	authCreds := &EmailCredentials{                 // 9
		Username: "myUsername",
		Password: "myPassword",
		Server:   "smtp.example.com",
		Port:     25,
	}
	auth := smtp.PlainAuth("",                      // 10
		authCreds.Username,
		authCreds.Password,
		authCreds.Server,
	)

	smtp.SendMail(                                  // 11
		authCreds.Server+":"+strconv.Itoa(authCreds.Port),
		auth,
		message.From,
		message.To,
		body.Bytes())
}
```