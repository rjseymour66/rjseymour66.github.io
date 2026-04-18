+++
title = 'Troubleshooting'
date = '2025-07-16T22:40:03-04:00'
weight = 5
draft = false
+++

## Context

A template accepts a context, which is data passed into the template. A template can be one of the following:
- simple value
- object
- method on an object


Usually, the current context is a the `Page` object. You can rebind the context to another value or object with `range` or `with` blocks. `with` is commonly used to change the context. For example, if you are within a shortcode but need to access the Page context, you can rebind it within the `with` block:

```go
{{ range slice "one" "two" "three" (dict "test" 1) }}
    <p>{{ . }}</p>
{{ end }}

// rebind to page
{{ with .Page }}
    <p>{{ . }}</p>
{{ end }}
```

`.Page` could have been any value, including a string. 

The context in a shortcode always refers to the shortcode itself, not the outer context. For example, to access the title in the `Page` context in a shortcode, you must use `.Page.Title` rather than `.Title`. In contrast, a layout page can refer to the same value with only `.Title`.

Access the outer context--in a shortcode, the `Page` context--by prepending the value with a `$`:

```go
{{ with "dogs" }}
    <p>{{ $.Page }} - {{ . }}</p> // [Page context] - dogs
{{ end }}
```

## Actions

A template action is any expression enclosed in double curly braces (`{{ }}`) that evaluates data, executes a function, controls the flow, etc.

## Variables

A variable is a user-defined value prepended with a `$`. It can hold any of the following:
- string
- integer
- floating point
- boolean
- slice
- map
- object

For slices, you can access a specific value with the `index` keyword:

```go
{{ $odds := slice 1 3 5 7 9}}
<p>Seconds index: {{ index $odds 2 }}</p>
```

Create a map with the `dict` keyword. You can access map values with dot notation:

```go
{{ $myMap := dict "one" 1 "two" 2 "three" 3 "four" 4 }}
<p>$myMap.two: {{ $myMap.two }}</p>
```

## Functions

There are a lot of functions. Use the alias when available.




## hugo-vals shortcode

<!-- {{< hugo-vals >}} -->


## page-methods

<!-- {{< page-methods >}} -->

## Resource methods

{{< resource >}}