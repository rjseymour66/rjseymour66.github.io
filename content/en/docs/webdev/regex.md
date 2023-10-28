---
title: "Regex"
weight: 40
description: >
  Regex cheatsheet.
---

## Links

- [PRACTICE SITE](https://regexr.com/)
- [Regex101 (practice)](https://regex101.com/)

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

> By default, a regex pattern returns only the first result that it finds. To return all, use the `/g` expression flag.

## Character sets

`[bcf]at` matches the following:
- bat
- cat
- fat

## Ranges

Ranges match any single character, digit, or symbol within the range. `[a-z]at` matches any word that begins with a lowercase letter and ends with `at`.

| Range type  | Example |
|:------------|:--------|
| Partial     | [a-f] or [g-p]  |
| Capitalized |  [A-Z] |
| Digit       | [0-9]  |
| Symbol      | [#$%&@]  |
| Mixed       | [a-zA-Z0-9]  |

## Repeating characters

Specify the number of repeating characters with the `{val}` syntax:

| Expression    | Match |
|:--------------|:--------|
| `a{5}`        | `aaaaa`  |
| `[a-z]{4}`    |  any four-letter lowercase word |
| `[a-z]{6,}`   | any lowercase word with 6 or more letters  |
| `[a-z]{8,11}` | any lowercase word with between 8 or 11 letters (inclusive) |
| `[0-9]{11}`       | 11-digit number |


## Metacharacters

Write compact regex with metacharacters:

| Expression    | Match |
|:--------------|:--------|
| `\d`          |  any digit, equal to `[0-9]`|
| `\w`       | any word character, such as a letter or digit. |
| `\W`    |  any character that is not a word character or digit. |
| `\s`   | any whitespace  |
| `\S` | anything other than a whitespace character. |
| `\t`       | tab character |

Combine these with repeating characters:

- `\w{5}`: five-letter word or number
- `\d{11}`


## Special characters

| Character  | Description | Example |
|:-----------|:--------|:------------|
| `+` | One or more of the previous characters.  | `c+at` matches `cat` or `ccccat` |
| `?` | Zero or one of the previous characters.  | `c?at` matches `cat` or `at` |
| `*` | Zero or more of the previous characters. | `c*at` matches `at` or `cat` or `cccat` |
| `\` | Escape character. | `\d*` matches `d*` |
| `[^]` | Negate notation. Do not match the characters after the `^` within the braces. | `b[^a]ld` matches `bold` but not `bald` |
| `.` | Match any digit, letter, or symbol except newline. | `.{8}` matches any eight-character token. |

For example:
- `.+` matches one or more unlimited number of characters.
- `[a-z]+` matches all lowercase words.


## Groups

Groups apply pattern matching to a section of the expression. `book(ing)?` matches `book` and `booking`.

## Alternate characters

Use the pipe (`|`) character to match one or the other option. For example:

- `bat|bit`

You can match the same values by enclosing the options in parentheses:
- `b(a|i)t`

## Starting and ending characters

> **REVISIT THIS**

| Character  | Description | Example |
|:-----------|:--------|:------------|
| `^` | Matches pattern at the start of the string. Place this at the beginning of the pattern. |  |
| `$` | Matches pattern at the end of the string. Place this at the end of the pattern. |  |

## Quantifiers

Quantifiers define how many 
- `+`: match one or more of the preceding token.
- `?`: optional. Optionally, you want to match the preceding token.
- `*`: match zero or more of the preceding token. Wildcard.
- `.`: match anything except a newline.
- `\.`: search for a period.
- `{min,max}`: match any characters between min and max. So `/\w{4,5}\/g` matches any 4 or 5 consequtive word characters.
- `[bc]at`: matches bat or cat. 
- `[a-zA-Z]`: any word that ends in `at` and starts with an uppercase or lowercase letter. Works with numbers too (`[0-9]`)
- `()`: groups

## Lookaheads

Read this [Sitepoint article](https://www.sitepoint.com/demystifying-regex-with-practical-examples/).