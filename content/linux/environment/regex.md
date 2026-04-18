+++
title = 'Regex'
date = '2025-09-07T18:57:18-04:00'
weight = 60
draft = false
+++


Regular expressions (regex) let you search, validate, and transform text using a compact pattern syntax. You write a pattern, and the regex engine matches it against a string to find specific characters, words, or structures.

Practice tools:
- [regexr.com](https://regexr.com/)
- [regex101.com](https://regex101.com/)

## How regex patterns work

Wrap your pattern in forward slashes. Everything between them is the pattern:

```
/cat/g
```

The `g` after the closing slash is a flag. Flags control how the match runs. The next section covers them in detail.

## Expression flags

A flag follows the closing slash and controls how the pattern applies. By default, regex returns only the first match it finds. Use `g` most often:

| Flag | Name             | Description                             |
| :--- | :--------------- | :-------------------------------------- |
| `g`  | Global           | Returns all matches, not just the first |
| `i`  | Case insensitive | Matches uppercase and lowercase         |
| `m`  | Multiline        | Treats each line as a separate string   |
| `s`  | Single line      | Makes `.` match newline characters      |
| `u`  | Unicode          | Enables full Unicode character matching |
| `y`  | Sticky           | Matches only at the current position    |

## Character sets

A character set matches any one character from a defined list. Place the options inside square brackets.

`[bcf]at` matches:
- `bat`
- `cat`
- `fat`

## Ranges

A range matches any single character within a span. `[a-z]at` matches any word that begins with a lowercase letter and ends with `at`:

| Range type | Example             |
| :--------- | :------------------ |
| Partial    | `[a-f]` or `[g-p]`  |
| Uppercase  | `[A-Z]`             |
| Digit      | `[0-9]`             |
| Symbol     | `[#$%&@]`           |
| Mixed      | `[a-zA-Z0-9]`       |

## Repeating characters

Control how many times a character or set repeats using `{n}` syntax:

| Expression    | Description                                      |
| :------------ | :----------------------------------------------- |
| `a{5}`        | `aaaaa`                                          |
| `[a-z]{4}`    | Any four-letter lowercase word                   |
| `[a-z]{6,}`   | Any lowercase word with 6 or more letters        |
| `[a-z]{8,11}` | Any lowercase word with between 8 and 11 letters |
| `[0-9]{11}`   | Any 11-digit number                              |


## Metacharacters

Metacharacters are shorthand for common character classes:

| Expression | Description                                       |
| :--------- | :------------------------------------------------ |
| `\d`       | Any digit, equivalent to `[0-9]`                  |
| `\w`       | Any word character (letter, digit, or underscore) |
| `\W`       | Any non-word character                            |
| `\s`       | Any whitespace character                          |
| `\S`       | Any non-whitespace character                      |
| `\t`       | A tab character                                   |

Combine metacharacters with repeating character syntax:
- `\w{5}` matches any five-character word or number sequence
- `\d{11}` matches any 11-digit number


## Special characters

Special characters extend what a pattern can match:

| Character | Description                                              | Example                                  |
| :-------- | :------------------------------------------------------- | :--------------------------------------- |
| `+`       | One or more of the preceding token                       | `c+at` matches `cat` and `ccccat`        |
| `?`       | Zero or one of the preceding token                       | `c?at` matches `cat` and `at`            |
| `*`       | Zero or more of the preceding token                      | `c*at` matches `at`, `cat`, and `cccat`  |
| `\`       | Escapes the next character                               | `\.` matches a literal period; `\d*` matches the literal string `d*` |
| `[^]`     | Negation: matches any character not listed after the `^` | `b[^a]ld` matches `bold` but not `bald`  |
| `.`       | Matches any character except a newline                   | `.{8}` matches any eight-character token |

For example, `.+` matches one or more of any character, and `[a-z]+` matches any sequence of consecutive lowercase characters.


## Groups

Parentheses group part of a pattern so you can apply a quantifier to the whole group.

For example, `book(ing)?` matches both `book` and `booking`. The `?` makes the entire group optional.

Groups also capture their match for reuse. Most regex tools number captured groups from left to right starting at `$1`, so you can reference them in a replacement string.

## Alternate characters

Use the pipe character (`|`) to match either option. For example, `bat|bit` matches `bat` or `bit`.

Enclose the alternating portion in parentheses to keep the pattern compact. For example, `b(a|i)t` matches the same values as `bat|bit`.

## Starting and ending characters

Anchors match a position in the string, not a character. Use `^` and `$` together to match an exact string with nothing before or after it. For example, `^cat$` matches only `cat` and does not match `the cat`, `cats`, or `cat sat`:

| Character | Description                     | Example                                              |
| :-------- | :------------------------------ | :--------------------------------------------------- |
| `^`       | Matches the start of the string | `^cat` matches `cat` in `cat sat` but not `the cat` |
| `$`       | Matches the end of the string   | `cat$` matches `cat` in `the cat` but not `cat sat` |

## Lookaheads

A lookahead matches a pattern only when it is followed (or not followed) by another pattern. The lookahead condition is not included in the match result.

A positive lookahead (`?=`) matches the preceding pattern only when followed by the lookahead condition. For example, the following pattern matches `100` in `100 dollars` but not in `100 euros`:

```bash
\d+(?= dollars)
```

A negative lookahead (`?!`) matches the preceding pattern only when it is not followed by the lookahead condition. For example, the following pattern matches `100` in `100 euros` but not in `100 dollars`:

```bash
\d+(?! dollars)
```