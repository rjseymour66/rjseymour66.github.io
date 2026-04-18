+++
title = 'Configuration files'
linkTitle = 'Config files'
date = '2025-08-14T11:40:02-04:00'
weight = 50
draft = false
+++

Config files let you supply persistent configuration without command-line arguments. Go supports JSON natively; YAML, INI, and environment variables require small third-party libraries or the standard `os` package.

## JSON

Go's `encoding/json` package reads JSON config files into structs. JSON doesn't support comments.

Here is `conf.json`:

```json
{
  "username": "rjs",
  "password": "secret",
  "port": 4001,
  "storage": "path/to/local/store",
  "enableFlag": true
}
```

This code sample parses `conf.json` and stores its values in memory:
1. Capitalize all `config` fields to export them.
2. `os.Open` returns a `*os.File` and an error. `*os.File` implements `io.Reader`.
3. Always `defer` the file close immediately after opening it.
4. `json.NewDecoder` takes an `io.Reader`.
5. `Decode` matches keys in `conf.json` to fields in the `config` struct. Matching is case-insensitive. Unmatched keys are ignored. Only exported fields are matched.

   {{< admonition "" tip >}}
   Use `Decode` rather than `json.Unmarshal` when reading from a stream, file, or connection. Use `json.Unmarshal` when the JSON data is already in memory as a `[]byte`.
   {{< /admonition >}}

This example writes errors to stderr with `fmt.Fprintf(os.Stderr, ...)`, which makes the destination explicit. In production code, prefer `log.Printf` (adds timestamps automatically) or `slog.Error` (Go 1.21+, structured key-value output). All three write to stderr by default.

```go
type config struct {                        // 1
	Username   string
	Password   string
	Port       int
	Storage    string
	EnableFlag bool
}

func jsonConfig() {
	file, err := os.Open("conf.json")       // 2
	if err != nil {
		fmt.Fprintf(os.Stderr, "cannot open conf.json: %v\n", err)
		return
	}

	defer file.Close()                      // 3
	decoder := json.NewDecoder(file)        // 4
	cfg := config{}
	err = decoder.Decode(&cfg)              // 5
	if err != nil {
		fmt.Fprintf(os.Stderr, "error parsing config file: %v\n", err)
		return
	}

	// output with field names
	fmt.Printf("%+v\n", cfg)
}
```


## YAML

YAML supports comments, which JSON does not. Go doesn't include a native YAML parser. The [Gypsy library](https://github.com/kylelemons/go-gypsy) provides a simple key-value API for reading YAML files:

- [Gypsy documentation](https://pkg.go.dev/github.com/kylelemons/go-gypsy/yaml)

For new projects, consider [`gopkg.in/yaml.v3`](https://pkg.go.dev/gopkg.in/yaml.v3). It's actively maintained and uses struct tags like `encoding/json`.

Here is `conf.yaml`:

```yaml
# Test conf file
username: "abc"
password: "secret"
port: 4001
enableFlag: true
```

This code sample parses `conf.yaml` and stores its values in memory:
1. The `yaml.ReadFile` function takes a string and returns a `*File`.
2. The `*File` type has methods to retrieve values of type `string`, `bool`, and `int`.
3. `GetInt` returns an `int64`, so you need to convert it to an `int` before passing it to `strconv.Itoa`.


```go
func yamlConfig() {

	cfg, err := yaml.ReadFile("conf/conf.yaml")         // 1
	if err != nil {
		fmt.Println(err)
		return
	}

	var username, password, port string
	var intPort int64
	var enableFlag bool

	username, err = cfg.Get("username")                 // 2
	if err != nil {
		fmt.Println("`username` flag not set", err)
		return
	}

	password, err = cfg.Get("password")
	if err != nil {
		fmt.Println("`password` flag not set", err)
		return
	}

	intPort, err = cfg.GetInt("port")                   // 3
	if err != nil {
		fmt.Println("`port` flag not set", err)
		return
	}
	port = strconv.Itoa(int(intPort))

	enableFlag, err = cfg.GetBool("enableFlag")
	if err != nil {
		fmt.Println("`enableFlag` flag not set", err)
		return
	}
}
```

## INI

Go doesn't include a native INI parser. The `gopkg.in/ini.v1` library is widely used:

- [Getting started](https://ini.unknwon.io/docs/intro/getting_started)
- [API reference](https://pkg.go.dev/gopkg.in/ini.v1)

Here is `conf.ini`:

```ini
; Top level comment
[user]
username = rjs
password = secret

[server]
port = 4001

[flags]
enable_flag = true
```

This code sample parses `conf.ini` and stores its values in memory:
1. `ini.Load` accepts one or more file paths and returns a `*ini.File` and an `error`.
2. Chain `Section` and `Key` calls to navigate the file hierarchy. `String` returns the value as a string. `Bool` returns a boolean and an error; it accepts `true`, `false`, `on`, `off`, `1`, and `0`.

```go
func iniConfig() {

	cfg, err := ini.Load("conf/conf.ini")
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	username := cfg.Section("user").Key("username").String()
	password := cfg.Section("user").Key("password").String()
	port := cfg.Section("server").Key("port").String()
	
    enableFlag, err := cfg.Section("flags").Key("enable_flag").Bool()
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

## Environment variables

Environment variables configure applications per deployment without touching the filesystem. They're the standard approach for containers and cloud environments, where filesystem access may be restricted or unavailable.

{{< admonition "12-factor apps" note >}}
Configuring applications with environment variables is one of the factors in a 12-factor app.
{{< /admonition >}}

You can set environment variables in `.bashrc` or export them directly to a shell session. Namespace your variables to avoid conflicts:

```bash
export MYAPP_PORT="4005"    # set env var in shell session
unset MYAPP_PORT            # unset env var
```

### os.Getenv

Use `os.Getenv` when a missing variable and an empty value are both invalid. It returns an empty string in both cases, so a single check handles either condition:

```go
func main() {
	port := os.Getenv("MYAPP_PORT")
	if port == "" {
		fmt.Fprintf(os.Stderr, "MYAPP_PORT is not set\n")
		os.Exit(1)
	}
	// use port
}
```

### os.LookupEnv

Use `os.LookupEnv` when you need to distinguish between a missing variable and one explicitly set to an empty string. It returns the value and a boolean that is `false` only when the variable is absent:

```go
func main() {
	port, ok := os.LookupEnv("MYAPP_PORT")
	if !ok {
		fmt.Fprintf(os.Stderr, "MYAPP_PORT is not set\n")
		os.Exit(1)
	}
	if port == "" {
		fmt.Fprintf(os.Stderr, "MYAPP_PORT is set but empty\n")
		os.Exit(1)
	}
	// use port
}
```
