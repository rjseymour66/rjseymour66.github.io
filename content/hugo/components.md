+++
title = 'Components'
date = '2025-07-20T12:13:19-04:00'
weight = 4
draft = false
[params]
  is_this_true = true
  key-value-pair = 'this is the value'
  [params.author]
    name = 'Jack Frost'
+++


<!-- {{< hugo-vals >}} -->

<!-- {{< page-methods >}} -->

<!-- ## Resource methods -->

<!-- {{< resource >}} -->

{{< admonition "Note" note >}}
Make sure you know how these changes affect you.
{{< /admonition >}}

{{< admonition "Warning" warning >}}
Make sure you know how these changes affect you.
{{< /admonition >}}

{{< admonition "Error" error >}}
Make sure you know how these changes affect you.
{{< /admonition >}}

{{< admonition "Tip" tip >}}
Make sure you know how these changes affect you.
{{< /admonition >}}



{{< colors >}}

## Code blocks

```go
{{ range slice "one" "two" "three" (dict "test" 1) }}
    <p>{{ . }}</p>
{{ end }}

// rebind to page
{{ with .Page }}
    <p>{{ . }}</p>
{{ end }}
```