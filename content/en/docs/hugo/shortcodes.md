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