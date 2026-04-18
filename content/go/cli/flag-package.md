+++
title = 'Flag Package'
date = '2025-08-13T13:15:11-04:00'
weight = 20
draft = false
+++

Arguments
: Necessary parameters for the program to run.

Flags
: Optional settings that modify the program behavior.

Define flags, and then execute their logic in the main method. Think of CLI flag implementations as programs that call external libraries, even if the library is included in the same project.

## FlagSet

When you create flags in Go, each flag definition is saved in a structure called [*Flagset](https://pkg.go.dev/flag#FlagSet) for tracking. The default Flagset for the flag package is named `CommandLine`, and it has access to all Flagset functions.

You can also create a custom FlagSet. This makes it easier to test and allows multiple parsers in the same program.

## Parse

The `Parse()` function extracts each command line flag in the Flagset and creates name/value pairs, where the name is the flag name, and the value is the argument provided to the flag. Next, it updates any command line flag’s internal variable.


## Types of flags

You have two options when defining flags:
- `flag.[Type]`: Returns a variable that holds the address where the flag value is stored. This is good for simple values.
- `flag.[Type]Var`: Accepts a pointer to the address where the flag value is stored. Use this when you want to use a configuration struct for your flag values, or you want to use Go's `flag` package to implement short and long names for a flag.

There is an implementations of these methods for each type. For example, there is a `flag.String`, `flag.StringVar`, `flag.Bool`, `flag.BoolVar`, etc.


| Feature                     | `flag.[Type]`   | `flag.[Type]Var` |
| :-------------------------- | :-------------- | :--------------- |
| Returns value?              | Yes             | No               |
| Requires existing variable? | No              | Yes              |
| Uses pointer?               | Returns pointer | You pass pointer |
| Cleaner for structs?        | No              | Yes              |


### flag.[Type]

This method returns an address where the flag's value is stored:
1. Define a variable.
2. Parse the flag in an `init` function
3. Dereference the variable to access its value.

This example creates a flag named `name` and outputs "Hello [name]" to STDOUT:

```go
var name = flag.String("name", "World", "A name to say hello to.")

func init() {
	flag.Parse()
}

func main() {
	fmt.Printf("Hello %s\n", *name)
}
```

### flag.[Type]Var

This method accepts a pointer to an existing variable. You pass a pointer to this method, and the value is stored in the variable when the flags are parsed.

Use this method when you want to use Go's flag package to create both long and short names for a single flag. When the compiler parses the flags, it assigns the value to whichever flag was passed to on the command line.

1. Define a variable
2. Pass a pointer to this method within the `init` function. Order matters---if the user passes both the long and short flag name, the compiler assigns the value to the flag defined last:
3. Parse the flags in the `init` function.

This example creates two flags: `n` and `name`. Both point to the `name` variable. The `main` method outputs "Hello [name]" to STDOUT:

```go
var name string

func init() {
	flag.StringVar(&name, "name", "World", "A name to say hello to.")
	flag.StringVar(&name, "n", "World", "A name to say hello to.")
	flag.Parse()
}

func main() {
	fmt.Printf("Hello %s\n", name)
}
```

### Custom flags

You can create custom flag value parsers beyond the built-in `flag.Var` and `flag.[Type]Var` parsers. This lets you create flag value parsers with additional features, such as argument validation. You have to implement the `Value` interface:
1. `String` is used when you print default values or help output.
2. `Set` parses the string input, validates it, and stores it.

```go
type Value interface {
    String() string 		// 1
    Set(string) error 		// 2
}
```


1. Create a new type so you can attach methods to it.
2. An adapter function that converts an `int` pointer to a `positiveIntValue` pointer.
3. You need to convert the argument to a `string`. This function dereferences `n`, converts `n` to an `int`, then converts it to a `string`.
4. This function verifies that the input is a positive `int`:
   1. Parse the string into an integer. `0` tells the compiler to detect the base size from the string.
   2. `IntSize` is a constant that determines the number of bytes based on your system (32 or 64).
   3. Check whether the argument is positive.
   4. If the argument is a positive `int`, set the caller to the argument.

```go
// config.go
type positiveIntValue int 								// 1

func asPositiveIntValue(p *int) *positiveIntValue { 	// 2
	return (*positiveIntValue)(p)
}

func (n *positiveIntValue) String() string {
	return strconv.Itoa(int(*n))
}

func (n *positiveIntValue) Set(s string) error { 		// 4
	v, err := strconv.ParseInt( 						
		s,
		0,  											// 4.1
		strconv.IntSize, 								// 4.2
	)
	if err != nil {
		return err
	}
	if v <= 0 { 										// 4.3
		return errors.New("should be greater than zero")
	}
	*n = positiveIntValue(v) 							// 4.4

	return nil
}
```

To implement the customer value parser, use the `Var` function, and caste the given variable as the custom value parser type:

```go
// config.go
func parseArgs(c *config, args []string) error {
	fs := flag.NewFlagSet("hit", flag.ContinueOnError)

	fs.StringVar(&c.url, "url", "", "HTTP server `URL` (required)")
	fs.Var(asPositiveIntValue(&c.n), "n", "Number of requests") 			// custom value parser
	fs.Var(asPositiveIntValue(&c.c), "c", "Concurrency level") 				// custom value parser
	fs.Var(asPositiveIntValue(&c.rps), "rps", "Requests per second") 		// custom value parser

	return fs.Parse(args)
}
```

{{< admonition "Default values" note >}}
You can't set a default value in the custom value parser definition. Instead, set the default value when you define the configuration object.
{{< /admonition >}}

## Flags vs positional arguments

A flag is an optional argument that customizes a tool's behavior. A Positional argument is a required value that define a tool's primary purpose.

This example parses flags and a mandatory `url` argument. When you use both flags and positional arguments, the flags are parsed first, and then you access the positional arguments by index with `fs.Arg()`:


1. Create a usage flag that uses flagset methods to print the following default help message: `<tool-name> [options] url`, and then prints the help text for the registered flags.
2. Register the flags.
3. Parse the flags and return any error.
4. Extracts the positional argument. After you call `Parse`, all remaining non-flag arguments are stored internally. You need to access them with `fs.Arg(index)`. The first argument is `fs.Arg(0)`, the second `fs.Arg(1)`, and so on. Here, we extract only the `url` field.

```go
// config.go
func parseArgs(c *config, args []string) error {
	fs := flag.NewFlagSet("hit", flag.ContinueOnError)
	fs.Usage = func() { 													// 1
		fmt.Fprintf(fs.Output(), "usage: %s [options] url\n", fs.Name())
		fs.PrintDefaults()
	}

	fs.Var(asPositiveIntValue(&c.n), "n", "Number of requests") 			// 2
	fs.Var(asPositiveIntValue(&c.c), "c", "Concurrency level")
	fs.Var(asPositiveIntValue(&c.rps), "rps", "Requests per second")

	if err := fs.Parse(args); err != nil { 									// 3
		return err
	}
	c.url = fs.Arg(0) 														// 4

	return nil
}
```

## Validating flags

FlagSet doesn't support validating mandatory arguments or flags, so you have to validate flag values after they are parsed and set in the configuration object. Creating a separate validation lets you check edge cases.

For example, the following function takes the `config` struct and validates its field values:
1. Verify the given URL is valid.
2. Verify the number of requests is less than the currency level.

```go
// config.go
func validateArgs(c *config) error {

	u, err := url.Parse(c.url) 													// 1
	if err != nil {
		return fmt.Errorf("invalid value %q for url: %w", c.url, err)
	}
	if c.url == "" || u.Host == "" || u.Scheme == "" {
		return fmt.Errorf(
			"invalid value %q for url: requires a valid url", c.url)
	}
	if c.n < c.c { 																// 2
		return fmt.Errorf(
			"invalid value %d for flag -n: should be greater than flag -c: %d", c.n, c.c)
	}
	return nil
}
```

Add `validateArgs` after the flags and positional arguments are parsed.
1. Validates the parsed flag values.
2. If there is an error, print the error to the FlagSet's output.
3. Print the FlagSet's usage message that is defined at the top of the function.
   
```go
func parseArgs(c *config, args []string) error {
	fs := flag.NewFlagSet("hit", flag.ContinueOnError)
	fs.Usage = func() {
		fmt.Fprintf(fs.Output(), "usage: %s [options] url\n", fs.Name())
		fs.PrintDefaults()
	}

	// Register flags... 

	if err := fs.Parse(args); err != nil {
		return err
	}
	c.url = fs.Arg(0)

	if err := validateArgs(c); err != nil { 		// 1
		fmt.Fprintln(fs.Output(), err) 				// 2
		fs.Usage() 									// 3
		return err
	}

	return nil
}
```

## flag functions

Call each of these functions after you call `flag.Parse()`:

| Function               | Description                                                                                                                                                                   | Common Use                                          |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `flag.PrintDefaults()` | Prints all registered flags with their default values and usage text. Runs automatically with `-h` or `--help` when using the default `FlagSet`. Can also be called manually. | Show built-in help/usage output.                    |
| `flag.VisitAll()`      | Runs a callback function for **every** registered flag (set or unset). Useful for custom help formatting or documentation generation.                                         | Build custom usage/help text.                       |
| `flag.Visit()`         | Runs a callback function only for flags that the **user actually set**.                                                                                                       | Log or process only the flags the user passed in.   |
| `flag.Arg(i int)`      | Returns the *i-th* positional argument (after all flags). Index is 0-based. Returns `""` if out of range.                                                                     | Access specific positional args (e.g., input file). |
| `flag.Args()`          | Returns a slice of all positional arguments (after all flags).                                                                                                                | Get all non-flag arguments for further processing.  |
| `flag.NArg()`          | Returns the number of positional arguments (after all flags).                                                                                                                 | Validate number of required positional args.        |
| `flag.NFlag()`         | Returns the number of flags explicitly set by the user.                                                                                                                       | Check if any flags were provided at all.            |


## Example

This example uses a config struct to define default settings for a custom flagset.

### Flagset configuration

1. `config` defines all the flags. When its implemented in `main`, you can define defaults.
2. `parseArgs` function is called from the CLI tool's entrypoint (`main`), so it returns an error.
3. `fs` is a custom flagset named "hit". Name the flagset after the program. `flag.ContinueOnError` defines the flagset's behavior---if parsing fails, it returns an error to the caller.
4. Define the flags in the flagset. The help flag (`-h`) is included by default.
   1. The backticks on the `url` definition tells Go to use that word as the argument name in the output. For example, the help is `-url URL` instead of `-url string`.
5. Parse the arguments and set the fields on `c`. It returns an error if there are issues with the flags.

```go
// config.go
type config struct { 													// 1
	url string
	n   int
	c   int
	rps int
}

func parseArgs(c *config, args []string) error { 						// 2
	fs := flag.NewFlagSet("hit", flag.ContinueOnError) 					// 3

	fs.StringVar(&c.url, "url", "", "HTTP server `URL` (required)") 	// 4
	fs.IntVar(&c.n, "n", c.n, "Number of requests")
	fs.IntVar(&c.c, "c", c.c, "Concurrency level")
	fs.IntVar(&c.rps, "rps", c.rps, "Requests per second")

	return fs.Parse(args) 												// 5
}
```

### Entrypoint

The entrypoint for the program does the following:
1. Initializes the configuration with default settings.
2. Calls `parseArgs` and exits on error. `parseArgs` takes a pointer to the config struct and the arguments passed to the program. The pointer lets the program mutate the struct, and `os.Args[1:]` omits the program name from the argument slice.
3. Prints the output message when you run the command.

```go
// hit.go
func main() {
	c := config{ 													// 1
		n: 100,
		c: 1,
	}

	if err := parseArgs(&c, os.Args[1:]); err != nil { 				// 2
		os.Exit(1)
	}

	fmt.Printf( 													// 3
		"%s\n\nSending %d requests to %q (concurrency: %d)\n",
		logo, c.n, c.url, c.c)
}
```