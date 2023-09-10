---
title: "Shortcodes"
linkTitle: "Shortcodes"
weight: 70
description: >
  How to add, work with, and process shortcodes.
---

Shortcodes are snippets of templates that wrap reusable HTML into functions. These functions are replaced with actual content at compile time.

## HTML shortcodes

Create an HTML shortcode with angle brackets (`< >`) enclosed in double curly braces (`{{ }}`).

## Markdown shortcodes

Create a markdown shortcode with percent signs (`% %`) enclosed in double curly braces (`{{ }}`). Hugo converts these to HTML.

This allows you to reuse content. You can create a shortcode file and reuse the contents anywhere that you use the Markdown shortcode.

## Inline shortcodes

Inline shortcodes are declared in the markdown content of the page and then used in that page. This cuts down on the amount of global shortcodes stored in the `shortcodes/` directory.

To use inline shortcodes, create a `config/_default/security.yaml` file and add the following configuration:

```yaml
enableInlineShortcodes: true
```

> You cannot nest inline shortcodes.

With the configuration in place, you can create a shortcode by appending the `.inline` keyword to the name of the shortcode. Then to reuse the shortcode, just add a single shortcode line:

```bash
# declare the shortcode
# {{% shortcodeName.inline %}}
This is the content.
# {{% /shortcodeName.inline %}}

# reuse the shortcode
# {{% shortcodeName.inline /%}}
```

## Templating

The following `price.html` shortcode parses a CSV file in the global `assets/` directory to display its price:

```go
{{- $product := default (.Get 0) (.Get "product") -}}   // 1
{{- if $product -}}   // 2
    {{- $products := resources.GetMatch "products.csv" -}}  // 3
    {{- $parsedProducts := $products | transform.Unmarshal (dict "delimiter" ",") -}}   // 4
    {{- range $r := $parsedProducts -}}   // 5
        {{- if eq (index $r 0) $product -}}   // 6
            $ {{- trim (index $r 2) " " -}}   // 7
        {{- end -}}
    {{- end -}}
{{- end -}}
```

Before we describe the steps in the shortcode, not the following:
- In a shortcode, the context (`.`) is always the shortcode itself.
- `{{- -}}` trims whitespace on either side of the string output. If you use just the left or right curly braces and endash, then it trims whitespace on only that side.

The preceding example does the following:

1. Assign the `$product` variable to either the first argument (by default), or the first value for a key named "product".
   
   The `Get` keyword accesses positional parameters within shortcodes. https://gohugo.io/functions/get/
2. If `$product` is not a zero value...
3. Look in the `assets/` directory (the default global resources folder) for a file named `products.csv`, and assign it to the `$products` variable.
4. Parse the file into a slice of slices, delimiting by the `,` character, and assign the outer slice to the variable `$parsedProducts`.
5. For each item in `$parsedProducts`...
6. If the first column for any row equals `$product`...
7. Print the 3rd column (0-indexed) for that row

### Inner

The `.Inner` function grabs the content within the opening and closing shortcode braces. For example, the following shortcode builds an unordered list. The number of list items is determined by the argument passed to the shortcode, and the text content of the li is determined by the content within the shortcode braces:

```go
{{$count := default 5 (default (.Get 0) (.Get "count"))}}
{{$inner := .Inner | markdownify}}
{{if gt $count 0}}
<ul>
    {{range seq $count}}
    <li>{{$inner}}</li>
    {{end}}
</ul>
{{end}}
```