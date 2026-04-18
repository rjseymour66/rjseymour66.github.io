+++
title = 'Strings'
date = '2025-10-03T00:24:34-04:00'
weight = 20
draft = false
+++

## Raw strings

Create a raw string with backticks in place of double quotes. Raw strings ignore all formatting and escape characters, allow double quotes, and let you create multi-line strings:

```go
var str = `
This is
a "multi-line" string
`
```

## Bytes and runes

In Go, a string is an immutable slice of bytes in UTF-8 encoding. UTF-8 is a variable-length encoding that allows one to four bytes. A byte is an alias for `int8`. Go stores a string as slice of UTF-8 encoded bytes so it can efficiently send strings across the network in its default format, or it can be converted to runes for character manipulation.

### rune

A rune is the type that Go uses in place of characters (`char`). A rune is an alias for `int32` and represents a Unicode code point, such as `U+0041` for the letter "A":

```go
var a = 'A'         // creates a rune of type int32
var a byte = 'A'    // creates a byte of type int8
```
Because each character in a string can use up to four bytes, you should convert the string to a slice of runes (`[]rune`) when you manipulate it. Otherwise, you need to know the length in bytes of each character to avoid corrupting the string. 

Follow these rules when you manipulate strings in Go:
- rune: When working with characters, such as replacing or slicing a string. Converting a string to a `[]rune` decodes the string into Unicode code points, which ensures that each index corresponds to a whole character regardless of the number of bytes it uses in UTF-8.
- byte: When working with ASCII characters and binary data.

## Building strings

Build a single string from multiple strings and data. All strings in this section return `The time is 9:45AM in the morning.`.

### Concatination

Concatinate strings with double quotes:

```go
var concat string = "The time is " + time.Now().Format(time.Kitchen) + " in the morning."
```

You can also concatenate strings, one operation at a time. This method is less performant than others because Go creates a new string with each operation, which requires more memory allocation:

```go
concat2 := "The time is "
concat2 += time.Now().Format(time.Kitchen)
concat2 += " in the morning"
```

### Join

`Join` in the `strings` package takes an array of strings and data, and a separator. It returns a single concatinated string separated by the given separator, so you don't have to pad the strings with a space:

```go
var join string = strings.Join([]string{
    "The time is",
    time.Now().Format(time.Kitchen),
    "in the morning."}, " ")
```


### Sprint and Sprintf

`Sprint` and `Sprintf` functions behave like the `Print[f]` functions, but they return a `string`. Use `Sprintf` for a formatted string:

```go
var spr string = fmt.Sprint("The time is ", time.Now().Format(time.Kitchen), " in the morning.")
var spf string = fmt.Sprintf("The time is %v in the morning.", time.Now().Format(time.Kitchen))
```

### Builder

Strings are immutable, and string concatenation is expensive in Go. A string builder is an idiomatic way to build strings. It accumulates bytes in its internal buffer without making additional memory allocations:

1. Create a new `builder` type.
2. Build the string with `WriteString` and the string that you want to add to the builder.
3. Extract the final string with the `String()` method.
   
```go
var builder strings.Builder                             // 1
builder.WriteString("The time is ")                     // 2
builder.WriteString(time.Now().Format(time.Kitchen))
builder.WriteString(" in the morning.")
var built string = builder.String()                     // 3
```

Here is a more complex example that implements the `String` interface on a URL type. It uses the `Grow` method to set the size of the Builder's internal buffer so it does not have to reallocate memory when it is full.

1. Create constants that represent the length of the URL separators.
2. Calculate the size of the URL using the constants. For example, `https://mydomain/resource-name`.
3. Create the Builder.
4. `Grow` sets the Builder's internal buffer size.
5. Write each part of the URL to the Builder.
6. Return the string.


```go
func (u *URL) String() string {
	if u == nil {
		return ""
	}

	const ( 									// 1
		lenSchemeSeparator = len("://")
		lenPathSeparator   = len("/")
	)
	lenURL := len(u.Scheme) +  					// 2
			  lenSchemeSeparator + 
			  len(u.Host) + 
			  lenPathSeparator + 
			  len(u.Path)

	var s strings.Builder 						// 3
	s.Grow(lenURL) 								// 4

	if sc := u.Scheme; sc != "" { 				// 5
		s.WriteString(sc)
		s.WriteString("://")
	}
	if h := u.Host; h != "" {
		s.WriteString(h)
	}
	if p := u.Path; p != "" {
		s.WriteByte('/')
		s.WriteString(p)
	}
	return s.String() 							// 6
}
```

## Splitting a string

### Cut

`Cut` splits a string using a separator. It returns the portions before and after the separator and a Boolean that indicates whether the separator was found. If it is not found, it returns the original string in the before portion.

In this example, the function parses a URL. It uses `strings.Cut` twice: once to separate the scheme from the host and path, and another to separate the host and path. It uses the [comma OK idiom](../../fundamentals/idioms/#comma-ok) to check whether the first call to `Cut` succeeds:

1. Call `Cut` to separate the scheme from the rest of the URL.
2. Comma-ok idiom to check that the separator existed and the URL is parseable.
3. Call `Cut` again to separate the host from the path. Ignore the boolean this time because not all URLs require a path portion.

```go
func Parse(rawURL string) (*URL, error) { 				
	scheme, rest, ok := strings.Cut(rawURL, "://") 	 	// 1
	if !ok { 											// 2
		return nil, errors.New("missing scheme")
	}

	host, path, _ := strings.Cut(rest, "/") 			// 3
	return &URL{
		Scheme: scheme,
		Host:   host,
		Path:   path,
	}, nil
}
```

### Split

{{< admonition "Newline and spaces" tip >}}
The output from `Split` includes newline characters. In addition, this function might return empty spaces if there are characters with multiple spaces.

To elminate spaces and newlines, use [Fields](#fields) or [FieldsFunc](#fieldsfunc).
{{< /admonition >}}

`Split` is helpful when you are working with data delimited by a separator, such as CSV. You can take input and split it into an array of strings.

This example splits up a raw string:

```go
array := strings.Split(orginal, " ")
fmt.Printf("%q", array)
// ["I" "loved" "her" "against" "reason," "against" "promise,"
// "\nagainst" "peace," "against" "hope," "against" "happiness,"
// "against" "\nall" "discouragement" "that" "could" "be."]
```

### SplitN

`SplitN` lets you split for an arbitrary number of elements, and then combines the remaining fields into a single string:

```go
array := strings.SplitN(orginal, " ", 8)
fmt.Printf("%q", array)
// ["I" "loved" "her" "against" "reason," "against" "promise,"
// "\nagainst peace, against hope, against happiness, against \nall discouragement that could be."]
```

### SplitAfter

`SplitAfter` preserves the delimiter in the output on all elements except the last one:

```go
array := strings.SplitAfter(orginal, " ")
fmt.Printf("%q", array)
// ["I " "loved " "her " "against " "reason, " "against " "promise, "
// "\nagainst " "peace, " "against " "hope, " "against " "happiness, "
// "against " "\nall " "discouragement " "that " "could " "be."]
```

### Fields

Fields splits strings by removing one or more consecutive spaces by calling the `unicode.IsSpace` helper internally:

```go
fields := strings.Fields(orginal)
fmt.Printf("%q", fields)
// ["I" "loved" "her" "against" "reason," "against" "promise,"
// "against" "peace," "against" "hope," "against" "happiness,"
// "against" "all" "discouragement" "that" "could" "be."]
```

To remove punctuation, use `FieldsFunc`.

### FieldsFunc

`FieldsFunc` works the same as fields, but it accepts a Boolean function that lets you extend the separator:
1. The function should return true when the parser reaches a character that you do not want in a field. Here, `f` returns `true` if the rune is a punctuation mark or not a letter. So when the Fields function sees a rune, it passes it to the function. If the function returns `false`, it starts a field and appends the rune. This continues until the function returns `true`, and then it ends that field and splits.
   
   For example, if it encounters a comma, the `isPunc` function returns `true` and that character is not considered part of the field. If it encounters a space, `isPunc` returns false, but `!unicode.IsLetter` returns `true`, so that is not considered part of the field.
2. Pass the helper to `FieldsFunc` to return the array.
```go
f := func(c rune) bool { 								// 1
	return unicode.IsPunct(c) || !unicode.IsLetter(c)
}

fields := strings.FieldsFunc(orginal, f) 				// 2
fmt.Printf("%q", fields)
```

## Trimming

These functions remove leading and trailing characters from a string.

### Trim

`Trim` takes a string and a cutset, which is a string that contains all the characters that you want to cut from the start and end of the string.

In the following example, we remove the parentheses, exclamation point, and any spaces. Notice how the exclamation point is removed even though it isn't the last character in the string: 
```go
bugs := "(And that's all, Folks!)"
cutset := "()! "
trimmed := strings.Trim(bugs, cutset) 	// That's all, Folks
```

### TrimLeft and TrimRight

`TrimLeft` and `TrimRight` remove the cutset characters from the left and right of the string, respectively:

```go
trimLeft := strings.TrimLeft(bugs, cutset) 		// That's all, Folks!)
trimRight := strings.TrimRight(bugs, cutset)  	// (That's all, Folks
```

### TrimFunc*

`TrimFunc`, `TrimFuncLeft`, and `TrimFuncRight` substitutes the cutset for a function that inspects the Unicode code points at the start and end of the string.

If the function returns `true`, `TrimFunc` removes the rune from the string. Here, the function returns `true` if the rune is a punctuation mark or not a letter:

```go
bugs := "(That's all, Folks!)"
f := func(c rune) bool {
	return unicode.IsPunct(c) || !unicode.IsLetter(c)
}
trimmed := strings.TrimFunc(bugs, f)  		// That's all, Folks
```

### TrimSpace

The most commonly trimmed characters are whitespace (`" "`, `\n`, `\t`, etc.), so Go provides a convenience method that removes these characters:

```go
bugs := "\n\n(That's all, Folks!)\r\t\n"
noSpace := strings.TrimSpace(bugs) 			// (That's all, Folks!)
```

### TrimPrefix and TrimSuffix

```go

```
## Substrings

A string is a slice of bytes, so you can slice it like it an array or slice collection to create a substring.

When you create a substring, do not manually count to find the starting and ending index of the substring you want to extract. Go uses UTF-8 encoding, which allocates 1-4 bytes for each character. If you manually count the string characters and slice the string, you might get unexpected results. Instead, use the following methods to create the substring:
1. Use `Index` to find the starting position of the substring.
2. Use the starting index and the length of the string to find the ending index.
3. Slice the string into a substring with the slicing syntax. Here, the character at the `start` index is included in the substring, but the character at `end` is not---the string is sliced directly before that character.

```go
start := strings.Index(orginal, "against reason") 	// 1
end := start + len("against reason")             	// 2
substr := orginal[start:end] 						// 3
```

### Contains

`Contains` accepts a source string a substring and returns a boolean that indicates whether the source string includes the given substring.

{{< admonition "" note >}}
`Contains` is a convenience method---its a wrapper around `Index`, which returns a `-1` if the substring is not in the source.
{{< /admonition >}}

Here, we compare `Contains` and `Index`:

```go
has := strings.Contains(orginal, "against") 	// true
index := strings.Index(orginal, "tester") 		// -1
```

### HasSuffix and HasPrefix

Check whether a string begins with a given substring with `HasSuffix` and `HasPrefix`. These return a boolean:

```go
prefix := strings.HasPrefix(orginal, "I loved her") 		// true
suffix := strings.HasSuffix(orginal, "that could be.") 		// true
```
### Count

Returns the number of times the given substring occurs in a source string:

```go
count := strings.Count(orginal, "against") 	// 6
```

## Replace* methods

You might want to replace portions of a string with another string. Use the `Replace*` methods. The following sections use this raw string as the starting string:

```go
orginal := `
I loved her against reason, against promise, 
against peace, against hope, against happiness, against 
all discouragement that could be.`
```

### Replace

This function accepts a string, the portion you want to replace, the new string that you want to replace the portion with, and the number of instances that you want to replace. Passing `-1` as the last parameters replaces all instances:

```go
replaceOne := strings.Replace(orginal, "against", "with", 1)
replaceTwo := strings.Replace(orginal, "against", "with", 2)
replaceAll := strings.Replace(orginal, "against", "with", -1)
```

These examples output the following:

```bash
# replaceOne
I loved her with reason, against promise, 
against peace, against hope, against happiness, against 
all discouragement that could be.

# replaceTwo
I loved her with reason, with promise, 
against peace, against hope, against happiness, against 
all discouragement that could be.

# replaceAll
I loved her with reason, with promise, 
with peace, with hope, with happiness, with 
all discouragement that could be.
```

### ReplaceAll

`ReplaceAll` is equivalent to `Replace` with `-1` as the final argument:

```go
replaceAll := strings.ReplaceAll(orginal, "against", "with")
// I loved her with reason, with promise, 
// with peace, with hope, with happiness, with 
// all discouragement that could be.
```

### Replacer

`Replacer` is a type that lets you make multiple string replacements at one time. If you need to replace more than one string, create and use a `Replacer` because it is more performant than multiple `Replace` calls:

1. Create a replacer. The factory function takes a list of strings, where each second string is replaced by its preceding string. In this example, `"her"` is replaced by `"him"`, and so on.
2. Call the replacer's `Replace` method, passing in the string that contains the words that you want to replace.

```go
replacer := strings.NewReplacer("her", "him", "against", "for", "all", "some") 	// 1
replaced := replacer.Replace(orginal) 											// 2
// I loved him for reason, for promise, 
// for peace, for hope, for happiness, for 
// some discouragement that could be.
```

## Conversions

### Bytes

#### To bytes

To convert a string to a slice of bytes, typecast the string as an slice of bytes:

```go
func main() {
	str := "abcABC"
	bytes := []byte(str)
	fmt.Println(bytes)      // [97 98 99 65 66 67]
```

#### From bytes

To convert a slice of bytes as a string, typecast the slice as a string:
```go
func main() {
    bytes := []byte{97, 98, 99, 65, 66, 67}
	str := string(bytes)
	fmt.Println(str)        // abcABC
}
```

### Numbers

The `strconv` package provides methods to convert strings to and from numbers.

#### To numbers

Useful when dealing with human-readable, formatted text like JSON. It is handy because its human-readable, but everything is a `string`.

Remember that the `Atoi` and `Parse*` functions read strings and return numbers.

##### Atoi

`Atoi` stands for "(A)lphanumeric (to) (i)nteger". It is a convenience function for `ParseInt`. It returns an integer and an error:

```go
atoi, _ := strconv.Atoi("12345") 	// 12345
```
##### ParseInt

`ParseInt` parses the string into an integer with additional options. The first parameter is the string, second parameter is the base (0, 2-64), and the final parameter is the bit size. This table summarizes possible bit size values and their corresponding Go type:

| Bit size | Type    |
| :------- | :------ |
| `0`      | `int`   |
| `8`      | `int8`  |
| `16`     | `int16` |
| `32`     | `int32` |
| `64`     | `int64` |

```go
pInt, _ := strconv.ParseInt("12345", 10, 0) 	// 12345
```

##### ParseFloat

`ParseFloat` parses a string into a `float` type. The second parameter specifies the precision. For example, this expression converts the string to a `float64`:

```go
f, _ := strconv.ParseFloat("1.2345", 64) 	// 1.2345
```

##### ParseBool

Use this when you need to parse an integer that represents a boolean value. It accepts the following values:

- True: `1`, `t`, `T`, `TRUE`, `true`, `True`
- False: `0`, `f`, `F`, `FALSE`, `false`, `False`

```go
b, _ := strconv.ParseBool("True") 	// true
```

#### From numbers

Many human-readable formats, such as JSON, need to be converted to strings before transmission over the network.

The functions in this section do not return errors because you can always make a number a string. Remember that the `Itoa` and `Format*` functions read numbers and return strings. 

##### Itoa

`Itoa` stands for "(I)nteger (to) (s)tring". It is a convenience function for `FormatInt`:

```go
itoa := strconv.Itoa(12345) 	// "12345"
```

##### FormatInt

`FormatInt` takes an `int64` and and a base between `2` and `36`. This means that you can convert strings to binary format:

```go
int64Str := strconv.FormatInt(int64(12345), 10) 	// "12345"
binStr := strconv.FormatInt(int64(12345), 2) 		// "11000000111001"
```

##### FormatFloat

Format float accepts the following parameters:
- `f`: Float that you want to format.
- `fmt`: format verb (`'f'`, `'e'`, `'E'`, `'g'`, `'G'`)
- `prec`: precision (number of digits)
  - `'f'`, `'e'`, `'E'`: number of digits after the decimal point
  - `'g'`, `'G'`: number of significant digits
  - `-1`: use the smallest number of digits necessary
- `bitSize`: `32` for `float32`, `64` for `float64`

This table summarizes the possible output formats:

| Example                                | Output       |
| -------------------------------------- | ------------ |
| `FormatFloat(1234.56789, 'f', 2, 64)`  | `1234.57`    |
| `FormatFloat(1234.56789, 'e', 4, 64)`  | `1.2346e+03` |
| `FormatFloat(1234.56789, 'E', 3, 64)`  | `1.235E+03`  |
| `FormatFloat(1234.56789, 'g', 6, 64)`  | `1234.57`    |
| `FormatFloat(1234.56789, 'f', -1, 64)` | `1234.56789` |


## Reading from STDIN

Command line tools that accept user input or output from a piped command need to read strings from stdin.

### Scan

Use `Scan` to read a single string with no spaces. You need to create a variable, and then pass `Scan` a pointer to that variable so it can change its value:

1. Create a `string` variable.
2. Pass `Scan` a pointer to the variable.
3. Handle any errors.

```go
func main() {
	var input string 									// 1
	fmt.Print("Please enter a word: ")
	n, err := fmt.Scan(&input) 							// 2
	if err != nil { 									// 3
		fmt.Println("error with user input:", err, n)
	} else {
		fmt.Println("You entered: ", input)
	}
}
```

{{< admonition "" note >}}
`Scan` can read data types other than `string`, such as numbers, etc.
{{< /admonition >}}

### ReadString

Use `ReadString` if you need to capture input with spaces, such as a sentence. 

1. Create a new reader from Stdin. `NewReader` wraps Stdin in a buffered reader for more efficient reads. The `bufio` Reader also has [many methods](https://pkg.go.dev/bufio#Reader) that helps reading input.
2. `ReadString` takes a delimiter, so this implementation reads from Stdin until it encounters an newline character. When you press the Enter key, the program moves to the next step.
3. Handles any errors.
   
```go
func main() {
	reader := bufio.NewReader(os.Stdin) 			// 1
	fmt.Print("Please enter input: ")
	input, err := reader.ReadString('\n') 			// 2
	if err != nil { 								// 3
		fmt.Println("error with user input:", err) 	
	} else {
		fmt.Println("You entered: ", input)
	}
}
```

## HTML escaping

You need to sanitize string input when you inject it into a webpage to prevent cross-site scripting (XSS) and HTML injection attacks.

The `EscapeString` method converts special characters into their HTML entity equivalents. The `UnescapeString` method does the opposite:

```go
str := `<p>Ampersand (&) and "quotes".</p>`
escaped := html.EscapeString(str)         // &lt;p&gt;Ampersand (&amp;) and &#34;quotes&#34;.&lt;/p&gt;
unescaped := html.UnescapeString(escaped) // <p>Ampersand (&) and "quotes".</p>
```

## Regex

Regular expressions (regex) is a notation for describing a search pattern in a string. The syntax is used across many languages, such as Perl and Python.

To use Go's `regex` package, perform these general steps:
1. Create a `Regexp` struct instance from your regex definition.
2. Call one of its `Find` methods to return a matching string or index.

All examples in this section use the following string:

```go
src := `I loved her against reason, against promise, 
against peace, against hope, against happiness, against 
all discouragement that could be.`
```

### Creating a regex

To create a regex, use either the `Compile` or `MustCompile` method. `Compile` returns an error, but `MustCompile` panics if there is an issue.

For the regex definition, use backticks. Regex patterns commonly use backslashes, which are interpreted differently with double quotes. This regex matches any string that starts with "against" and has any word after it:

```go
re, err := regexp.Compile(`against [\w]+`)
must := regexp.MustCompile(`against [\w]+`)
```
After you create the regex, you can use it to find matches.


### MatchString

Returns a boolean that indicates if the regex has any matches in the string:

```go
re := regexp.MustCompile(`against [\w]+`)
matchString := re.MatchString(src) 	// true
```

### FindString

Returns the matching string:

```go
re := regexp.MustCompile(`against [\w]+`)
findString := re.FindString(src) 	// against reason
```

### FindAllString

Takes two arguments: the source string, and the number of matches that you want returned. To return all matches, enter a negative number (`-1`):

```go
re := regexp.MustCompile(`against [\w]+`)
findAllString := re.FindAllString(src, -1)
fmt.Printf("%q\n", findAllString)
// ["against reason" "against promise" "against peace" "against hope" "against happiness"]
```

### FindStringIndex

Finds indices for the first match and returns them in a slice. The first integer indicates is the starting index, and the second integer is the ending index:
1. Get the slice of integers.
2. Use the indices in the slice to extract a matching string.

```go
re := regexp.MustCompile(`against [\w]+`)
locs := re.FindStringIndex(src) 		// [12, 26]
locsMatch := src[locs[0]:locs[1]] 		// against reason
```

### FindAllStringIndex

Returns a two-dimensional slice of the start and end indices for all matching strings:

```go
re := regexp.MustCompile(`against [\w]+`)
allLocs := re.FindAllStringIndex(src, -1)
// [[12 26] [28 43] [46 59] [61 73] [75 92]]
```

### ReplaceAllString

Replaces all matches in the `src` string with the second argument:

```go
re := regexp.MustCompile(`against [\w]+`)
replaced := re.ReplaceAllString(src, "anything")
// I loved her anything, anything, 
// anything, anything, anything, against 
// all discouragement that could be.
```

### ReplaceAllStringFunc

This method accepts as its second parameter a function that runs on each match.

For example, you can make all matches uppercase:

```go
re := regexp.MustCompile(`against [\w]+`)
replacedUpper := re.ReplaceAllStringFunc(src, strings.ToUpper)
// I loved her AGAINST REASON, AGAINST PROMISE, 
// AGAINST PEACE, AGAINST HOPE, AGAINST HAPPINESS, against 
// all discouragement that could be.
```

You can also define a higer-order function that performs custom mutations:
1. The method must take a string argument and return a string. This function capitalizes the second word in the match ("reason").
2. Pass the function to `ReplaceAllStringFunc` as the second parameter. Do not add parentheses (`()`) after the function. This is because adding `()` means you get the result of the function. `ReplaceAllStringFunc` takes only a function with a specific signature, and it calls that function internally.

```go
re := regexp.MustCompile(`against [\w]+`)
f := func(in string) string {  							// 1
	split := strings.Split(in, " ")
	split[1] = strings.ToUpper(split[1])
	return strings.Join(split, " ")
}
replacedCustom := re.ReplaceAllStringFunc(src, f) 		// 2
// I loved her against REASON, against PROMISE, 
// against PEACE, against HOPE, against HAPPINESS, against 
// all discouragement that could be.
```

### All methods

The following table summarizing the available methods. Here are some common patterns in the function names that help you understand their behavior:
- `*All`: Return all matches in a string, depending on `n`.
- Without `*All`: Return only the first match.
- `*String`: Returns strings or slices of strings.
- Without `*String`: Returns an array of bytes (`[]byte`).
- `*Index`: Returns the index of the match.

| Method                           | Description                                                                                              |
| -------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `regexp.Compile`                 | Compiles a regular expression into a `*Regexp` object. Returns an error if the expression is invalid.    |
| `regexp.MustCompile`             | Like `Compile`, but panics if the expression is invalid. Commonly used for constant regex patterns.      |
| `regexp.Match`                   | Checks if the byte slice matches the regex pattern (no need to compile). Returns a boolean and an error. |
| `regexp.MatchString`             | Checks if the string matches the regex pattern (no need to compile). Returns a boolean and an error.     |
| `(*Regexp).Match`                | Reports whether the regex matches the given byte slice.                                                  |
| `(*Regexp).MatchString`          | Reports whether the regex matches the given string.                                                      |
| `(*Regexp).Find`                 | Returns the first match of the regex in the byte slice, or `nil` if no match is found.                   |
| `(*Regexp).FindString`           | Returns the first match of the regex in the string, or an empty string if no match is found.             |
| `(*Regexp).FindAll`              | Returns all matches (up to `n`) of the regex in the byte slice. Use `-1` for all matches.                |
| `(*Regexp).FindAllString`        | Returns all matching substrings (up to `n`) in the string.                                               |
| `(*Regexp).FindSubmatch`         | Returns a slice of byte slices: the full match followed by any capturing group matches.                  |
| `(*Regexp).FindSubmatchIndex`    | Returns a slice of index pairs identifying the start and end of the full match and submatches.           |
| `(*Regexp).FindAllSubmatch`      | Returns all matches and their submatches in a byte slice.                                                |
| `(*Regexp).FindAllSubmatchIndex` | Returns all match and submatch indices.                                                                  |
| `(*Regexp).ReplaceAll`           | Replaces all matches in the byte slice with the replacement text.                                        |
| `(*Regexp).ReplaceAllString`     | Replaces all matches in the string with the replacement text.                                            |
| `(*Regexp).ReplaceAllFunc`       | Replaces all matches using a function that computes each replacement.                                    |
| `(*Regexp).ReplaceAllStringFunc` | Replaces all matches in the string using a function that computes each replacement.                      |
| `(*Regexp).Split`                | Splits a string into substrings around matches of the regex. `n` limits the number of substrings.        |
| `(*Regexp).SubexpNames()`        | Returns the names of the capturing groups in the regex (if any).                                         |
| `(*Regexp).NumSubexp()`          | Returns the number of capturing groups in the regex.                                                     |
