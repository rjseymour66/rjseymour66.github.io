---
title: "Regex"
weight: 40
description: >
  Regex cheatsheet.
---
[PRACTICE SITE](https://regexr.com/)

Regex is a way to search through the text to validate text, find and replace, etc.

## Format

Start and end with a `/`. Everything within the forward slash is the regex:

```
/cat/g
```

## Expression flags

The `g` in `/cat/g` is an _expression flag_. An expression flag defines the scope of the regex search. The expression flags are listed below:
- `g`: global
- `i`: case insensitive
- `m`: multiline
- `s`: single line
- `u`: unicode
- `y`: sticky

Generally, you will use the `g` flag the most.

## Quantifiers

Quantifiers define how many 
- `+`: match one or more of the preceding token.
- `?`: optional. Optionally, you want to match the preceding token.
- `*`: match zero or more of the preceding token. Wildcard.
- `.`: match anything except a newline.
- `\.`: search for a period.
- `\w`: any word character, such as a letter.
- `\W`: any character that is not a word character.
- `\s`: any whitespace
- `\S`: anything other than a whitespace character.
- `{min,max}`: match any characters between min and max. So `/\w{4,5}\/g` matches any 4 or 5 consequtive word characters.
- `[bc]at`: matches bat or cat. 
- `[a-zA-Z]`: any word that ends in `at` and starts with an uppercase or lowercase letter. Works with numbers too (`[0-9]`)
- `()`: groups