+++
title = 'Cobra'
date = '2025-08-13T15:51:46-04:00'
weight = 30
draft = false
+++

[Cobra](https://github.com/spf13/cobra) is a CLI framework for Go. It provides POSIX-style commands with automatic help text, shell completion, and flag management. Use Cobra when your tool needs subcommands, persistent flags, or shell completion. For simple single-command tools, the standard [`flag` package](../flag-package/) is sufficient.

## Install

Cobra has two components:

- **`cobra` library**: The framework your application imports as a dependency.
- **`cobra-cli`**: A code generator that scaffolds commands and subcommands.

Add the library to your module:

```bash
go get github.com/spf13/cobra@latest
```

Install the generator binary to your `$GOPATH/bin`:

```bash
go install github.com/spf13/cobra-cli@latest
```

## Project setup

### Initialize a project

1. Create a project directory and initialize a Go module:

   ```bash
   mkdir files
   cd files
   go mod init github.com/username/files
   ```

2. Initialize a Cobra project:

   ```bash
   cobra-cli init
   ```

   This creates the following structure:

   ```
   files
   ├── cmd
   │   └── root.go
   ├── go.mod
   ├── go.sum
   ├── LICENSE
   └── main.go
   ```

3. Verify the setup:

   ```bash
   go run . --help
   ```

### Understand the generated files

**`main.go`** is the entry point. It delegates entirely to the `cmd` package:

```go
package main

import "github.com/username/files/cmd"

func main() {
    cmd.Execute()
}
```

**`cmd/root.go`** defines the root command. It's the top-level command that all subcommands attach to. Edit it to match your project:

```go
package cmd

import (
    "os"

    "github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
    Use:   "files",
    Short: "A file management tool",
    Long:  `files lists and searches files on your system.`,
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}

func init() {
    // Register persistent flags and subcommands here.
}
```

## Configure cobra-cli

When `cobra-cli` generates new files, it populates the author name and license from a `.cobra.yaml` file in your home directory:

```yaml
author: First Last <name@email.com>
license: MIT
useViper: false
```

Set `useViper: true` if you want `cobra-cli` to scaffold [Viper](https://github.com/spf13/viper) configuration integration in every generated command file.

## Add a command

Add a command with `cobra-cli add`. The argument is the command name:

```bash
cobra-cli add list
```

`cobra-cli` creates `cmd/list.go` with boilerplate using `Run`:

```go
package cmd

import (
    "fmt"

    "github.com/spf13/cobra"
)

var listCmd = &cobra.Command{
    Use:   "list",
    Short: "A brief description of your command",
    Long:  `A longer description...`,
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("list called")
    },
}

func init() {
    rootCmd.AddCommand(listCmd)
}
```

Edit the file to update the descriptions and change `Run` to `RunE`:

```go
var listCmd = &cobra.Command{
    Use:   "list",
    Short: "List files in a directory",
    Long:  `List all files in the specified directory.`,
    RunE: func(cmd *cobra.Command, args []string) error {
        fmt.Println("list called")
        return nil
    },
}
```

The key fields in `cobra.Command`:

| Field   | Description                                                                       |
| :------ | :-------------------------------------------------------------------------------- |
| `Use`   | The command name and argument syntax shown in help text.                          |
| `Short` | A one-line description shown in the parent command's help listing.                |
| `Long`  | The full description shown when the user runs `files list --help`.                |
| `RunE`  | The function that runs when the command executes. Returns an error to the caller. |

Use `RunE` instead of `Run`. `RunE` returns an error, which Cobra prints and uses to set the exit code. `Run` silently discards errors.

The `init()` function wires the command to its parent with `AddCommand`. Every generated command does this automatically.

Run the command:

```bash
go run . list
# list called
```

## Add subcommands

Subcommands nest under a parent command. To add `files search name` and `files search type`, first add the parent:

```bash
cobra-cli add search
```

Then add the subcommands with `-p` to specify the parent command's variable name:

```bash
cobra-cli add name -p searchCmd
cobra-cli add type -p searchCmd
```

The `-p` flag tells `cobra-cli` which variable to call `AddCommand` on. Open `cmd/search.go` to confirm the generated variable name is `searchCmd`.

The generated `init()` in each subcommand file wires it to `searchCmd`:

```go
// cmd/name.go
func init() {
    searchCmd.AddCommand(nameCmd)
}

// cmd/type.go
func init() {
    searchCmd.AddCommand(typeCmd)
}
```

`cmd/search.go` wires `searchCmd` to `rootCmd`:

```go
// cmd/search.go
func init() {
    rootCmd.AddCommand(searchCmd)
}
```

The resulting command tree:

```
files
├── list
└── search
    ├── name
    └── type
```

Run a subcommand:

```bash
go run . search name
# name called
```

## Flags

### Persistent vs. local flags

Cobra supports two flag scopes:

- **Persistent flags**: Available to the command and all its subcommands. Register with `PersistentFlags()`.
- **Local flags**: Available only to the specific command. Register with `Flags()`.

```go
// Available to all commands under rootCmd
rootCmd.PersistentFlags().BoolP("verbose", "v", false, "Enable verbose output")

// Available only to listCmd
listCmd.Flags().BoolP("all", "a", false, "Include hidden files")
```

### Bind flags to variables

Binding flags directly to package-level variables avoids calling `GetString` or `GetBool` inside `RunE`, which each return an error you'd need to handle. Use the `VarP` variants to register both a long name (`--dir`) and a short name (`-d`):

```go
var listDir string
var listAll bool

func init() {
    rootCmd.AddCommand(listCmd)
    listCmd.Flags().StringVarP(&listDir, "dir", "d", ".", "Directory to list")
    listCmd.Flags().BoolVarP(&listAll, "all", "a", false, "Include hidden files")
}
```

### Require a flag

Mark a flag as required with `MarkFlagRequired`. Cobra returns an error automatically if the user omits it. No manual validation needed:

```go
listCmd.Flags().StringVarP(&listDir, "dir", "d", "", "Directory to list (required)")
listCmd.MarkFlagRequired("dir")
```

## Positional arguments

Use the `Args` field to validate positional arguments. Cobra provides built-in validators:

| Validator               | Description                                        |
| :---------------------- | :------------------------------------------------- |
| `cobra.NoArgs`          | Fails if any argument is provided.                 |
| `cobra.ExactArgs(n)`    | Requires exactly `n` arguments.                    |
| `cobra.MinimumNArgs(n)` | Requires at least `n` arguments.                   |
| `cobra.MaximumNArgs(n)` | Allows at most `n` arguments.                      |
| `cobra.RangeArgs(m, n)` | Requires between `m` and `n` arguments.            |
| `cobra.ArbitraryArgs`   | Accepts any number of arguments (default).         |

When the argument count is wrong, Cobra prints a descriptive error and the usage message automatically.

```go
var searchCmd = &cobra.Command{
    Use:   "search <directory>",
    Short: "Search for files in a directory",
    Args:  cobra.ExactArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        directory := args[0]
        fmt.Fprintf(cmd.OutOrStdout(), "Searching in: %s\n", directory)
        return nil
    },
}
```

## Test commands

Test Cobra commands by redirecting output to a `bytes.Buffer`, calling `Execute()`, and inspecting the buffer. Use `SetOut` and `SetErr` to inject test writers. This is the same `io.Writer` injection pattern used in the [`flag` package tests](../cli-test/).

```go
// cmd/helpers_test.go
package cmd

import (
    "bytes"
)

func executeCommand(args ...string) (string, string, error) {
    outBuf := new(bytes.Buffer)
    errBuf := new(bytes.Buffer)

    rootCmd.SetOut(outBuf)
    rootCmd.SetErr(errBuf)
    rootCmd.SetArgs(args)

    err := rootCmd.Execute()
    return outBuf.String(), errBuf.String(), err
}
```

{{< admonition "Shared state between tests" note >}}
Do not use `t.Parallel()` on tests that share the package-level `rootCmd`. Two problems arise with parallel tests: `SetArgs`, `SetOut`, and `SetErr` mutate shared state and will race; and package-level flag variables (such as `listDir` and `listAll`) are not reset between `Execute()` calls, so one test's flags leak into the next. Run command integration tests sequentially, or construct a fresh command tree per test.
{{< /admonition >}}

Create testdata so your tests have files to read:

```bash
mkdir cmd/testdata
touch cmd/testdata/foo.go cmd/testdata/bar.md
```

```go
// cmd/list_test.go
package cmd

import (
    "testing"
)

func TestListCommand(t *testing.T) {
    out, _, err := executeCommand("list", "--dir", "testdata")
    if err != nil {
        t.Fatalf("list: got error %v", err)
    }
    if out == "" {
        t.Error("list: stdout is empty, want file names")
    }
}

func TestListCommandMissingDir(t *testing.T) {
    _, _, err := executeCommand("list", "--dir", "/nonexistent-cobra-test-dir")
    if err == nil {
        t.Fatal("list: expected error for missing directory, got nil")
    }
}
```

Run the tests:

```bash
go test ./cmd/...
```

## Real-world example

This section shows a complete `files` CLI with `list` and `search` commands.

### Project layout

```
files
├── cmd
│   ├── helpers_test.go
│   ├── list.go
│   ├── list_test.go
│   ├── root.go
│   ├── search.go
│   ├── search_test.go
│   └── testdata
│       ├── foo.go
│       └── bar.md
├── go.mod
├── go.sum
└── main.go
```

### `cmd/root.go`

```go
package cmd

import (
    "os"

    "github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
    Use:   "files",
    Short: "A file management tool",
    Long:  `files lists and searches files on your system.`,
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}
```

### `cmd/list.go`

```go
package cmd

import (
    "fmt"
    "os"
    "strings"

    "github.com/spf13/cobra"
)

var (
    listDir string
    listAll bool
)

var listCmd = &cobra.Command{
    Use:   "list",
    Short: "List files in a directory",
    Long:  `List all files in the given directory. Pass --all to include hidden files.`,
    RunE: func(cmd *cobra.Command, args []string) error {
        entries, err := os.ReadDir(listDir)
        if err != nil {
            return fmt.Errorf("reading %q: %w", listDir, err)
        }
        for _, entry := range entries {
            if !listAll && strings.HasPrefix(entry.Name(), ".") {
                continue
            }
            fmt.Fprintln(cmd.OutOrStdout(), entry.Name())
        }
        return nil
    },
}

func init() {
    rootCmd.AddCommand(listCmd)
    listCmd.Flags().StringVarP(&listDir, "dir", "d", ".", "Directory to list")
    listCmd.Flags().BoolVarP(&listAll, "all", "a", false, "Include hidden files")
}
```

### `cmd/search.go`

```go
package cmd

import (
    "fmt"
    "os"
    "path/filepath"
    "strings"

    "github.com/spf13/cobra"
)

var searchExt string

var searchCmd = &cobra.Command{
    Use:   "search <directory>",
    Short: "Search for files in a directory",
    Long:  `Search recursively for files in the given directory. Filter by extension with --ext.`,
    Args:  cobra.ExactArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        root := args[0]
        return filepath.WalkDir(root, func(path string, d os.DirEntry, err error) error {
            if err != nil {
                return err
            }
            if d.IsDir() {
                return nil
            }
            if searchExt != "" && !strings.HasSuffix(d.Name(), "."+searchExt) {
                return nil
            }
            fmt.Fprintln(cmd.OutOrStdout(), path)
            return nil
        })
    },
}

func init() {
    rootCmd.AddCommand(searchCmd)
    searchCmd.Flags().StringVarP(&searchExt, "ext", "e", "", "Filter by file extension (e.g. go, md)")
}
```

### Usage

```bash
# List files in the current directory
files list

# List all files, including hidden ones
files list --all

# List files in a specific directory
files list --dir /tmp

# Search recursively for all Go files
files search . --ext go

# Search for Markdown files
files search /home/user/docs --ext md

# Search with no filter — returns all files
files search /home/user/docs
```

### Tests

```go
// cmd/search_test.go
package cmd

import (
    "strings"
    "testing"
)

func TestSearchCommand(t *testing.T) {
    out, _, err := executeCommand("search", "testdata")
    if err != nil {
        t.Fatalf("search: got error %v", err)
    }
    if !strings.Contains(out, "foo.go") {
        t.Errorf("search: output %q does not contain foo.go", out)
    }
}

func TestSearchCommandWithExt(t *testing.T) {
    out, _, err := executeCommand("search", "testdata", "--ext", "go")
    if err != nil {
        t.Fatalf("search --ext go: got error %v", err)
    }
    if !strings.Contains(out, "foo.go") {
        t.Errorf("search --ext go: output %q does not contain foo.go", out)
    }
    if strings.Contains(out, "bar.md") {
        t.Errorf("search --ext go: output should not contain bar.md")
    }
}

func TestSearchMissingArg(t *testing.T) {
    _, _, err := executeCommand("search")
    if err == nil {
        t.Fatal("search: expected error for missing argument, got nil")
    }
}
```
