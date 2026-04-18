+++
title = 'Configuration objects'
date = '2025-11-27T12:25:28-05:00'
weight = 10
draft = false
+++

The Functional Options pattern creates clean and flexible configuration objects. It lets you set values on a configuration object for your application. You create a configuration object with default values, and then you can optionally set custom values with the Functional Options pattern.

## Configuration struct

To begin, define your configuration struct. This example creates a configuration object for a CLI app that writes output and errors:

```go
type CliConfig struct {
	ErrStream, OutStream io.Writer
}
```

## Option function type

Define the `Option` type. This type is a function that accepts a pointer to a configuration object and returns an error:

```go
type Option func(*CliConfig) error
```

## Option constructors

Create the `Option` constructor methods. Idiomatic Go begins these functions with `WithXxx`.

These methods accept a value that you want to set for the config struct field and return an `Option` function. In other words, the parameter is equal to the field definition that you want to set in the config struct. `Option` takes a pointer to a configuration struct, so the value you pass to method is set in the config struct:

```go
func WithErrStream(errStream io.Writer) Option {
	return func(c *CliConfig) error {
		c.ErrStream = errStream
		return nil
	}
}

func WithOutStream(outStream io.Writer) Option {
	return func(c *CliConfig) error {
		c.OutStream = outStream
		return nil
	}
}
```

## Configuration constructor

The constructor method for the configuration object accepts a variable number of `Option` functions, and returns a config object and an error:
1. Create a config object with default settings.
2. Range over the `Option` arguments.
3. Set the option on the config object.
4. If there is an error, return an empty config object and the error.
5. Return the config object with default settings or any optional settings.

```go
func NewCliConfig(opts ...Option) (CliConfig, error) {  
	c := CliConfig{                         // 1
		ErrStream: os.Stderr,
		OutStream: os.Stdout,
	}

	for _, opt := range opts {              // 2
		if err := opt(&c); err != nil {     // 3
			return CliConfig{}, err         // 4
		}
	}
	return c, nil                           // 5
}
```

## Passing to an app

When you define your application, pass any data and a `CliConfig` object:

```go
func app(s []string, cfg CliConfig) { 
    //...
}
```

Here is how you use it in `main` with the defaults:

```go
func main() {
	words := os.Args[1:]
	if len(words) == 0 {
		fmt.Fprintln(os.Stderr, "No words provided.")
		os.Exit(1)
	}

	cfg, err := NewCliConfig()
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error creating config: %v\n", err)
		os.Exit(1)
	}

	app(words, cfg)
}
```

## Testing

To test the configuration, you need to mock the `OutStream` and `ErrStream`:
1. Mock the config options with a `bytes.Buffer`.
2. Initialize the `CliConfig` object, passing the option constructor functions as arguments.
3. Pass the test `config` to the test `app`.

```go
func TestMain(t *testing.T) {
	var stdoutBuf, stderrBuf bytes.Buffer                       // 1
	config, err := NewCliConfig(
        WithOutStream(&stdoutBuf), WithErrStream(&stderrBuf)    // 2
    )
	if err != nil {
		t.Fatal("Error creating config:", err)
	}

	app([]string{"main", "rick", "golang", "error"}, config)    // 3
    // ...
}
```