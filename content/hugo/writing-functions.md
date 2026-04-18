+++
title = 'Writing Functions'
date = '2025-07-13T09:12:21-04:00'
weight = 20
draft = false
+++


Here are the general steps to create a function using your content and Hugo built-ins. Here are some general tips:

- `printf` or `scratch` for formatting and intermediate variables
- `with`, `if`, and `range` to conditionally render blocks
- `.Site` for global data, `.Params` for front matter

1. Start with the data:
   - `.Content`: rendered HTML
   - `.RawContent`: raw markdown source
   - `.Title`, `.Date`, `.Params`: common page fields

2. Apply functions with pipeline syntax:
   ```go
   {{ $words := countwords .Content }}
   {{ $readingTime := div $words 250 }}
   ```
3. Use Hugo built-in functions:
   - `countwords`: counts words
   - `len`: gets length of lists/maps/strings
   - `add`, `div`, `mul`, `sub`: math
   - `math.Round`, `math.Floor`, `math.Ceil`
   - `time`, `now`, `dateFormat`
   - `where`, `index`, `delimit`, `replace`

For example, here we show the number of days since the page was last updated:

```go
{{ $days := div (sub now.Unix .Lastmod.Unix) 86400 }}
<p>Updated {{ math.Floor $days }} days ago</p>
```