---
title: "JSON data"
weight: 40
description: >
  How to work with JSON data.
---

> **IMPORTANT**: Always pass pointers to `json.Marshall` and `json.Unmarshall`. If you pass a value, the JSON functions operate on a copy and sends the data to the garbage collector after the function returns.


## Marshal into JSON

Marshalling transforms an in-memory data representation (i.e. a struct) into the JSON format for storage or transmission over the network.

### Struct tags

When you marshal data into JSON objects, you have to map the struct fields to the corresponding JSON object field. Use struct tags to associate a struct field to a field in the JSON object.

Struct tags are enclosed in backticks (\`\`). For mulit-word fields, use snake case: `json:"json_field"`. Always capitalize the struct field name to export it. Otherwise, Go's native [`encoding/json`](https://pkg.go.dev/encoding/json) package cannot access it. For example:

```go
type log struct {
	Level   string `json:"level"`
	Message string `json:"message"`
}
```
Struct tags also provide options to hide or omit fields:
- `-` (hyphen): When you do not want a struct field to appear in output.
- `omitempty`: Do not create output for the field if the value is empty (the zero value for the field).

For example:

```go
type Movie struct {
	ID        int64     `json:"id"`
	CreatedAt time.Time `json:"-"`
	Title     string    `json:"title"`
	Year      int32     `json:"year,omitempty"`
	Runtime   int32     `json:" runtime,omitempty"`
	Genres    []string  `json:"genres,omitempty"`
	Version   int32     `json:"version"`
}
```

### Examples

Your application uses a simple logger, and you want to store the log messages in JSON format. The logs use the following format:

```go
type log struct {
	Level   string
	Message string
}
```

You want to store the logs in the following JSON format:

```json
{
	"level": "Level",
	"message": "This is the log message",
}
```
#### Go struct to JSON encoding

If you have a struct that represents an object, then you can encode it in JSON formatting with the `encoding/json` package. The following steps encode a `log` object into a JSON format, which is a series of bytes that represent JSON encoding:

1. Add struct tags to the `log` defintion to associate the struct fields to the JSON object keys:
   ```go
	type log struct {
		Level   string `json:"level"`
		Message string `json:"message"`
	}
   ```
2. Import the `encoding/json` package and marshall the object. The `.Marshal()` method accepts a pointer to a JSON object, and it returns a slice of `bytes` and an `error`:
   ```go
	inMemLog := log{
		Level:   "Debug",
		Message: "This is the log message.",
	}

	jsonLog, err := json.Marshal(&inMemLog)
	if err != nil {
		fmt.Println(err)
	}

	// work with jsonLog
   ```
   Currently, `jsonLog` is a series of bytes. You can verify this by printing it:
   ```go
	fmt.Println(jsonLog) // [123 34 108 101 118 ...]
   ```

   To print it in a human-readable format, cast it as a string:
   ```go
	fmt.Println(string(jsonLog)) // {"level":"Debug","message":"This is the log message."}
   ```

### TODO

In some circumstances, you might not have the `log` object in memory. For example, you might be reading from the log service REST API. You can create a method on the `jsonLog` type that returns the marshalled data.

You must implement a method on the `jsonLog` type that mimics the [`Marshal()`](https://pkg.go.dev/encoding/json#Marshal) method:

1. Implement a `Marshall()` method on a pointer reciever:
   ```go
	func (j *jsonLog) MarshalJSON() ([]byte, error) {
		resp := jsonLog {
			Level: j.Level,
			Message: 
		}
	}
   ```


Create a MarshallJSON() method when you have to map structs to complex JSON objects. For example:

```go
func (r *todoResponse) MarshallJSON() ([]byte, error) {
	resp := struct {
		Results      todo.List `json:"results"`
		Date         int64     `json:"date"`
		TotalResults int       `json:"total_results"`
	}{
		Results:      r.Results,
		Date:         time.Now().Unix(),
		TotalResults: len(r.Results),
	}

	return json.Marshal(resp)
}
```
The preceding method models a response with an anonymous struct, then returns the response in JSON format.

## Unmarshal from JSON

When you unmarshal a JSON object, you are converting the JSON-encoded bytes into an in-memory object representation, such as a struct.

### Example

In the following example, your simple CRM application reads JSON-formatted data from the network in the following format:

```json
[
	{"name": "Steve", "age": 21},
	{"name": "Bob", "age": 68}
]
```

You need to parse each JSON object into the following struct:
```go
type person struct {
	Name string
	Age  int   
}
```
> NOTE: When you unmarshall data, you do not need to add struct tags. Remember to export (capitalize) each field in the struct.

1. For demonstration purposes, this slice of bytes mimics an array of two JSON objects that might arrive over the network:
   ```go
	jsonData := []byte(`[
	{"name": "Steve", "age": 21},
	{"name": "Bob", "age": 68}
	]`)

   ```
2. Create a data structure to store the unmarshalled `jsonData`:
   ```go
   var people []person
   ```
3. Unmarshal the data into the slice:
   ```go
	if err := json.Unmarshal(jsonData, &people); err != nil {
		fmt.Printf("Error: %v", err)
	}
   ```
Go unmarshals as many complete JSON objects as it can from the slice of bytes.

## Encoding JSON to a stream

The [json.Encoder](https://pkg.go.dev/encoding/json@go1.19.4#Encoder) accepts an `io.Writer`, and it encodes a struct in JSON format.

Use the [json.Encoder](https://pkg.go.dev/encoding/json@go1.19.4#Encoder) and its `.Endcode()` method to write the program's memory representation of JSON (a struct) to an output stream (an `io.Writer`). `Encoder` accepts the writer, and `.Encode()` takes the values that you want to encode. You can chain the methods. For example:

```go
var buffer bytes.Buffer

objToJSON := struct {
    Value string `json:"value"`
}{
    Value: stringName
}

if err := json.NewEncoder(&buffer).Encode(item); err != nil {
    // handle error
}
```

## Decoding JSON from stream

The [json.Decoder](https://pkg.go.dev/encoding/json@go1.19.4#Decoder) is just like the `json.Encoder`, except that it reads from an input stream (`io.Reader`) and decodes JSON into its memory representation (a struct).

The following example reads from a JSON-formatted file (an `io.Reader`) and decodes the JSON into a slice:

```go
f, err := os.Open(filePath)
if err != nil {
	return nil, err
}
defer f.Close()

var sl []SliceType

if err := json.NewDecoder(f).Decode(&sl); err != nil {
	return nil, err
}
```

## JSON in responses

JSON data is text. A basic JSON response:
- Marshals the JSON
- Sets at least the `Content-Type: application/json`
- Writes the bytes to the `writer`.

The `json.Marshal(v)` function is preferable to the `json.Encode(v)` function because the `Encode` function writes to the `io.writer` immediately, so you will not have time to add headers.

The following example creates a helper function that writes JSON then returns JSON in a healthcheck handler:

```go
func (app *application) writeJSON(w http.ResponseWriter, status int, data any, headers http.Header) error {
	js, err := json.Marshal(data)
	if err != nil {
		return err
	}

	js = append(js, '\n')

	for key, value := range headers {
		w.Header()[key] = value
	}

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	w.Write(js)

	return nil
}
```

Use this helper function with a `map` (or struct) that represents the JSON values:

```go
func (app *application) healthcheckHandler(w http.ResponseWriter, r *http.Request) {

	data := map[string]string{
		"status":      "available",
		"environment": app.config.env,
		"version":     version,
	}

	err := app.writeJSON(w, http.StatusOK, data, nil)
	if err != nil {
		app.logger.Print(err)
		http.Error(w, "The server encountered a problem and could not process your request", http.StatusInternalServerError)
	}
}
```

## JSON in requests

When you receive a request for JSON data, you have to decode the JSON into Go structs. Additionally, you should check the following:
- Request size is not too large
- There are no unknown fields in the request
- Errors are identified and receive appropriate responses
- There is only one JSON object sent in the request

The following function completes all these checks:

```go
func (app *application) readJSON(w http.ResponseWriter, r *http.Request, dst any) error {

	maxBytes := 1_048_576
	// limit size of req body
	// https://pkg.go.dev/net/http#MaxBytesReader
	r.Body = http.MaxBytesReader(w, r.Body, int64(maxBytes))

	// create new decoder
	dec := json.NewDecoder(r.Body)
	// no unknown fields
	dec.DisallowUnknownFields()

	err := dec.Decode(dst)
	if err != nil {
		var syntaxError *json.SyntaxError
		var unmarshalTypeError *json.UnmarshalTypeError
		var invalidUnmarshalError *json.InvalidUnmarshalError
		var maxBytesError *http.MaxBytesError

		// triage errors
		switch {
		case errors.As(err, &syntaxError):
			return fmt.Errorf("body contains badly-formed JSON (at character %d)", syntaxError.Offset)

		case errors.Is(err, io.ErrUnexpectedEOF):
			return errors.New("body contains badly-formed JSON")

		case errors.As(err, &unmarshalTypeError):
			if unmarshalTypeError.Field != "" {
				return fmt.Errorf("body contains incorrect JSON type for field %q", unmarshalTypeError.Field)
			}
			return fmt.Errorf("body contains incorrect JSON type (at character %d)", unmarshalTypeError.Offset)

		case errors.Is(err, io.EOF):
			return errors.New("body must not be empty")

		case strings.HasPrefix(err.Error(), "json: unknown field"):
			fieldName := strings.TrimPrefix(err.Error(), "json: unknown field ")
			return fmt.Errorf("body contains unknown key %s", fieldName)

		case errors.As(err, &maxBytesError):
			return fmt.Errorf("body must not be larger than %d bytes", maxBytesError.Limit)

		case errors.As(err, &invalidUnmarshalError):
			panic(err)

		default:
			return err
		}
	}

	// call Decode again to make sure the request body contained only
	// a single JSON value.
	err = dec.Decode(&struct{}{})
	if err != io.EOF {
		return errors.New("body must ony contain a single JSON value")
	}
	return nil
}
```