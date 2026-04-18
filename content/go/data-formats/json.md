+++
title = 'JSON'
date = '2025-10-11T17:08:03-04:00'
weight = 20
draft = false
+++

JavaScript Object Notation (JSON) is a lightweight data exchange format that is popular with RESTful web services and configuration. It is human-readable but also easily read by machines. It is described by [RFC 7159](https://datatracker.ietf.org/doc/html/rfc7159) and [ECMA-404](https://ecma-international.org/publications-and-standards/standards/ecma-404/).

## Marshal vs Unmarshal

| Action    | Meaning                         | Direction |
| --------- | ------------------------------- | --------- |
| Marshal   | Make (marshall) data to send    | Go → JSON |
| Unmarshal | Undo marshaling (read raw data) | JSON → Go |


## Decoding JSON

Decoding is another way to say "parsing". When you decode data, you convert it from an encoded format to the format you use in your program. You are parsing the data, byte by byte, into your program's memory.

### Unmarshalling vs decoding

The method you use to parse your JSON depends on the source. In short, `json.unmarshal` is best for in-memory JSON-formatted data, and a decoder works best when streaming data:

| Feature                                              | `json.Unmarshal`                                | `json.Decoder`                  |
| ---------------------------------------------------- | ----------------------------------------------- | ------------------------------- |
| Input type                                           | `[]byte`                                        | `io.Reader`                     |
| Reads                                                | Entire JSON at once                             | Streamed (chunk by chunk)       |
| Suitable for                                         | Small/medium data                               | Large or continuous data        |
| Can decode multiple JSONs                            | ❌                                               | ✅ Yes (via repeated `Decode()`) |
| Fine-grained control (unknown fields, numbers, etc.) | ❌ Limited                                       | ✅ More options                  |
| Common usage                                         | Parsing HTTP response body after `io.ReadAll()` | Streaming from file/socket/pipe |

### Bytes vs streams

Here is a JSON-formatted file, where each object is in a single array:

```json
[
  {
    "name": "Luke Skywalker",
    "height": "172",
    "mass": "77",
    "hair_color": "blond",
    "skin_color": "fair",
    "eye_color": "blue",
    "birth_year": "19BBY",
    "gender": "male",
    // ...
  },
  {
    "name": "C-3PO",
    "height": "167",
    "mass": "75",
    "hair_color": "n/a",
    "skin_color": "gold",
    "eye_color": "yellow",
    "birth_year": "112BBY",
    "gender": "n/a",
    // ...
  }
]
```

For comparison, this data is representative of a JSON stream. Each object is listed in no particular format, one after another:

```bash
{
    "name": "Luke Skywalker", 
    "height": "172", 
    "mass": "77", 
    "hair_color": "blond", 
    "skin_color": "fair", 
    "eye_color": "blue", 
    "birth_year": "19BBY", 
    "gender": "male", 
    ...
}
{
    "name": "C-3PO", 
    "height": "167", 
    "mass": "75", 
    "hair_color": "n/a", 
    "skin_color": "gold", 
    "eye_color": "yellow", 
    "birth_year": "112BBY", 
    "gender": "n/a", 
    ...
}
```

### Unmarshalling byte arrays

This example uses [this endpoint in SWAPI](https://swapi.dev/api/people/1), the Star Wars API.

Parsing JSON data involves two main steps:
1. Create a struct that models the JSON data you are decoding.
2. Unmarshal the raw JSON bytes into the struct.

The `Person` struct models the JSON object. Each field definition is followed by a struct tag, a raw string literal that maps that field to a key in the JSON for the encoder or decoder. A struct tag is always in the format `json:"key_name"`, with no space after the colon. Note that Go field names use camelCase, and struct tags use snake case:

```go
type Person struct {
	Name      string    `json:"name"`
	Height    string    `json:"height"`
	Mass      string    `json:"mass"`
	HairColor string    `json:"hair_color"`
	SkinColor string    `json:"skin_color"`
	EyeColor  string    `json:"eye_color"`
	BirthYear string    `json:"birth_year"`
	Gender    string    `json:"gender"`
	Homeworld string    `json:"homeworld"`
	Films     []string  `json:"films"`
	Species   []string  `json:"species"`
	Vehicles  []string  `json:"vehicles"`
	Starships []string  `json:"starships"`
	Created   time.Time `json:"created"`
	Edited    time.Time `json:"edited"`
	URL       string    `json:"url"`
}
```
The `unmarshal` function demonstrates how you can convert a JSON-formatted file from raw JSON bytes to a struct:
1. Open a local file. The file is a Reader.
2. `ReadAll` accepts a reader and returns a slice of bytes and an error. Here, we pass it the file handle.
3. Create a variable of type `Person`.
4. Call `json.Unmarshal` to unmarshal the slice of bytes in `data` into `person`. `Unmarshal` takes a slice of bytes to read and a pointer to a variable that can store the slice of bytes. It converts the raw bytes into the Go types defined by the pointer type.
5. Return the `person` variable.

```go
func unmarshal() Person {
	file, err := os.Open("person.json")                     // 1
	if err != nil {
		log.Println("Error opening json file: ", err)
	}
	defer file.Close()

	data, err := io.ReadAll(file)                           // 2
	if err != nil {
		log.Println("Error reading json data: ", err)
	}

	var person Person                                       // 3
	err = json.Unmarshal(data, &person)                     // 4
	if err != nil {
		log.Println("Error unmarshalling json data:", err)
	}
	return person                                           // 5
}
```

The `unmarshalAPI` function demonstrates how you can convert a GET request from raw JSON bytes to a struct. This method works when you are reading a single finite resource. For reading streams, see [Parsing JSON streams](#parsing-json-streams):
1. Makes a GET request to the given URL. This function returns an `http.Response` (a ReadCloser) and an `error`.
2. `ReadAll` accepts a Reader and returns a byte slice and an error.
3. Create a variable of type `Person`.
4. Call `json.Unmarshal` to unmarshal the slice of bytes in `data` into `person`. `Unmarshal` takes a slice of bytes to read and a pointer to a variable that can store the slice of bytes. It converts the raw bytes into the Go types defined by the pointer type.
5. Return the `person` variable.

```go
func unmarshalAPI() Person {
	res, err := http.Get("https://swapi.dev/api/people/1")      // 1
	if err != nil {
		log.Println("Cannot get from URL", err)
	}
	defer res.Body.Close()

	data, err := io.ReadAll(res.Body)                           // 2
	if err != nil {
		log.Println("Error reading json data: ", err)
	}

	var person Person                                           // 3
	err = json.Unmarshal(data, &person)                         // 4
	if err != nil {
		log.Println("Error unmarshalling json data:", err)
	}
	return person                                               // 5
}
```

Finally, call the functions. Here, we log the returned `Person` struct to the console:

```go
func main() {
	file := unmarshal()
	api := unmarshalAPI()

	fmt.Println(file)
	fmt.Println(api)
}
```

### Decoding streams

A stream of JSON data is a continuous flow of JSON-encoded bytes. Parsing a stream of data requires different techniques than parsing a JSON file, because file techniques require that you read the entire file into memory before parsing. When you parse a stream, you parse it in chunks.

You can decode a JSON stream with `json.Decoder` and channels. The `decode` function uses channels to decode the JSON stream:
1. Takes as an argument a channel
2. Open a local file, but it can open any Reader. The file is a Reader.
3. Create a new Decoder. A Decoder is a wrapper around a Reader. Here, we wrap `file`.
4. Create an infinite `for` loop that decodes `Person` objects. Another option is to use a while-style `for` loop and the `More` method. `More` returns a boolean that reports whether there is another element in the stream, so you wouldn't need to check for an EOF. For a working example, see the [Go json docs](https://pkg.go.dev/encoding/json#Decoder.Decode).
5. Create a `person` variable of type `Person`.
6. `Decode` reads the next JSON-encoded value from the Reader. `Decode` mutates `person`, so you need to pass it a pointer.
7. When `Decode` reaches the end of the stream, it returns `io.EOF`. Exit the loop with `break` when this occurs.
8. Handle any errors and `break`, if necessary.
9. Send the `person` value into the channel. This send channel is unbuffered, so it blocks until another goroutine retrieves its value.
10. Close the channel after the loop exits when it either completes or returns an error.

```go
func decode(p chan Person) {                                    // 1
	file, err := os.Open("stream.txt")                          // 2
	if err != nil {
		log.Println("Error opening JSON file: ", err)
	}
	defer file.Close()

	decoder := json.NewDecoder(file)                            // 3

	for {                                                       // 4
		var person Person                                       // 5
		err = decoder.Decode(&person)                           // 6
		if err == io.EOF {                                      // 7
			break
		}
		if err != nil {                                         // 8
			log.Println("Error decoding json data: ", err)
			break
		}
		p <- person                                             // 9
	}
	close(p)                                                    // 10
}
```

The `main` function runs `decode` in a goroutine. This means that `main` can retrieve data from a channel:

1. Create a channel of type `Person`.
2. Run `decode` in its own goroutine.
3. Create an infinite loop to read from the channel. Because `decode` is running in a separate thread, `main` can start reading from the channel immediately.
4. Check for values with the [comma-ok](../fundamentals/idioms#comma-ok) idiom. `person` receives any value from the `p` channel, and `ok` reports whether the channel is open or closed. It returns `true` if the channel is open and `false` if the channel is closed. This occurs if `decode` exited its loop because of error or reaching an EOF.
5. If `ok` is `true`, there is a value in the channel. Here, we print it to the console with the 3rd-party [pretty](github.com/kr/pretty) package.
6. If `ok` is `false`, break. There is no cleanup because `decode` closes the channel.

```go
func main() {
	p := make(chan Person)                  // 1
	go decode(p)                            // 2

	for {                                   // 3
		person, ok := <-p                   // 4
		if ok {                             // 5
			fmt.Printf("%# v\n", pretty.Formatter(person))
		} else {                            // 6
			break
		}
	}
}
```

## Encoding JSON

When you encode json, you create JSON-encoded data from in-memory data, such as a struct.

### Marshaling vs encoding

The method you use to convert application data into raw JSON streams depends on the source. `json.marshal` is best for in-memory data, and an `Encoder` works best when you want to write directly to the stream:

| Feature         | `json.Marshal`                                                            | `json.Encoder.Encode`                                                       |
| --------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Returns**     | `[]byte` (in memory)                                                      | writes directly to stream                                                   |
| **Destination** | You choose what to do with the bytes                                      | Goes straight to an `io.Writer`                                             |
| **Use case**    | You need the JSON data as bytes (e.g. for HTTP response body, file write) | You want to stream JSON directly (e.g. stdout, file, socket)                |
| **Performance** | Slightly more memory overhead (allocates a byte slice)                    | More efficient for streaming large data                                     |
| **Formatting**  | You can use `MarshalIndent` for pretty output                             | You can’t directly indent (though you can wrap the writer with an indenter) |
| **Newline**     | No newline automatically                                                  | Automatically adds newline after each object                                |


### Marshaling byte arrays

Marshaling JSON means taking a Go struct and converting it back into raw JSON bytes. First, the `get` function retrieves a resource and unmarshals the `[]byte` response into a `Person` object:

1. Get the `Person` resource at the given url.
2. Read the entire response into a byte slice. `data` is raw JSON data in memory.
3. Create a variable of type `Person`.
4. Unmarshal the raw bytes into the `person` variable.
5. Return `person`.

```go
func get(url string) Person {
	r, err := http.Get(url)                             // 1
	if err != nil {
		log.Println("Cannot get from URL", err)
	}
	defer r.Body.Close()                            

	data, err := io.ReadAll(r.Body)                     // 2
	if err != nil {
		log.Println("Error reading json data:", err)
	}

	var person Person                                   // 3
	json.Unmarshal(data, &person)                       // 4
	return person                                       // 5
}
```

In `main`, we use `get` to populate the `person` variable, then we convert the struct back into raw JSON data:
1. Retrieve the raw JSON data and store it in memory.
2. Marshal the data stored in `person` into raw JSON bytes. `MarshalIndent` is like a helper function for `Marshal`---in addition to the `person` struct, it accepts as variables a prefix and a level of indent. Here, we do not add a prefix and add an indent of one space.
3. `WriteFile` writes writes the data to `han.json`. For more information, see [Writing files](./fundamentals/input-output/#writefile).

```go
func main() {
	person := get("https://swapi.dev/api/people/14")        // 1

	data, err := json.MarshalIndent(&person, "", " ")       // 2
	if err != nil {
		log.Println("Cannot marshal person:", err)
	}
	err = os.WriteFile("han.json", data, 0644)              // 3
	if err != nil {
		log.Println("Cannot write to file", err)
	}
}
```


### Encoding streams

If you output a continual stream of data, you can stream a series of structs as raw JSON directly to to a Writer:
1. Build the URL with the `n` parameter. This function fetches the resource with an ID of `n`.
2. Read the response body with `ReadAll`, and store it in `data`. `ReadAll` accepts a Reader.
3. Create a variable of type `Person`.
4. Unmarshal the raw bytes into the `person` variable.
5. Return `person`.

```go
func get(n int) Person {
	r, err := http.Get("https://swapi.dev/api/people/" + strconv.Itoa(n))   // 1
	if err != nil {
		log.Println("Cannot get from URL", err)
	}
	defer r.Body.Close()

	data, err := io.ReadAll(r.Body)                     // 2
	if err != nil {
		log.Println("Error reading json data", err)
	}

	var person Person                                   // 3
	json.Unmarshal(data, &person)                       // 4
	return person                                       // 5
}
```

Next, create the stream of raw JSON data:
1. Create a new Encoder. An Encoder is a wrapper around a Writer. Here, the Writer is `stdout`.
2. Create a loop that increments a variable from 1 to 4.
3. Pass the variable to the `get` function during each iteration.
4. `Encode` immediately streams the raw JSON to `stdout` rather than writing it to memory first. This is different from [Encoding JSON](#encoding-json), where the JSON data was marshaled into a variable and then written to a file. An `Encoder` can stream directly to the underlying Writer.


```go
func main() {
	encoder := json.NewEncoder(os.Stdout)   // 1
	for i := 1; i < 4; i++ {                // 2
		person := get(i)                    // 3
		encoder.Encode(person)              // 4
	}
}
```
## Omitting fields

In some cases, not all fields of the JSON object are populated for each resource. You can omit fields that don't have data with the `omitempty` tag:

```go
type Person struct {
	Name      string    `json:"name"`
	Height    string    `json:"height"`
	...
	Species   []string  `json:"species,omitempty"`
	Vehicles  []string  `json:"vehicles,omitempty"`
	Starships []string  `json:"starships,omitempty"`
	...
}
```
Now, if fields with `omitempty` do not have data, you will not see them in the output:

```json
{
 "name": "Han Solo",
 "height": "180",
 "mass": "80",
 "hair_color": "brown",
 "skin_color": "fair",
 "eye_color": "brown",
 "birth_year": "29BBY",
 "gender": "male",
 "homeworld": "https://swapi.dev/api/planets/22/",
 "films": [
  "https://swapi.dev/api/films/1/",
  "https://swapi.dev/api/films/2/",
  "https://swapi.dev/api/films/3/"
 ],
 "starships": [
  "https://swapi.dev/api/starships/10/",
  "https://swapi.dev/api/starships/22/"
 ],
 "created": "2014-12-10T16:49:14.582Z",
 "edited": "2014-12-20T21:17:50.334Z",
 "url": "https://swapi.dev/api/people/14/"
}
```