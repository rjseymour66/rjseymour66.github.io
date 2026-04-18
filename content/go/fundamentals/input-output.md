+++
title = 'Input/output'
date = '2025-10-10T12:30:05-04:00'
weight = 50
draft = false
+++

In Go, input and output center around the `io.Reader` and `io.Writer` interfaces. A type that implements `io.Reader` is a "reader", and a type that implements `io.Writer` is a "writer". Here is a summary of each:
- Reader: A type that reads its own bytes. Each reader has a `Read` method that reads the contents of the reader itself and stores it in a slice of bytes in memory.
- Writer: A type that can receive bytes. Each writer has a `Write` method that writes a slice of bytes from memory into the writer itself.

{{< admonition "Memory management" note >}}
Memory management is a primary design feature of the Reader and Writer interface. Both interfaces require that the caller provide a byte slice (`[]byte`). This lets the caller allocate memory for one byte slice, read or write data into that slice, then do something with the data. The caller can fill that single buffer as many times as needed instead of allocating multiple byte slices.
{{< /admonition >}}

## io.Reader

A reader is a source you can pull bytes from. Data flows from the stream through the reader into a buffer in memory:

```
stream -> Reader -> buffer
```

The `io.Reader` interface has one method:

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Any type that implements `Read` is a reader. `Read` fills `p` with up to `len(p)` bytes and returns how many were read. When the source is exhausted, it returns `io.EOF`.

| Use case | Reader | Buffered reader |
|:---|:---:|:---:|
| Simple, small reads | ✅ | ❌ |
| Line-by-line reading | ❌ | ✅ |
| Minimize syscalls | ❌ | ✅ |
| Real-time reads (no delay) | ✅ | ❌ |
| Parsing or tokenizing | ❌ | ✅ |


### Unbuffered

`Read` reads data directly from the input stream, one read at a time with no buffering. Each call requires a system call, so use `Read` when reading a small or known amount of data and you need control over how many bytes are read at a time.

Use unbuffered I/O in the following circumstances:

| Use case | Example |
|:---|:---|
| Small files directly | Use `os.Open()` + `Read()` for config files or metadata `file.Read(buf)` |
| From in-memory data | Use `strings.NewReader("data")` or `bytes.NewReader()` when you already have the content in memory. |
| From a network connection | Use `conn.Read()` to process packets or headers directly from a TCP stream. |
| Stdin directly (single read) | `os.Stdin.Read(buf)` for reading a fixed-size input or when buffering isn’t needed. |
| Composing custom readers | Implement `io.Reader` to wrap or transform streams (e.g., decrypting reader, counting reader). |

A file in Go is a reader because the [File interface](https://pkg.go.dev/io/fs#File) implements `Read`. This example reads a small file into a buffer and prints its contents:

1. Open the file. The file is a reader.
2. Create a buffer that can hold 1KB.
3. `Read` reads bytes from the file into the buffer.
4. Convert the buffer from a slice of bytes to a string.
5. Output the contents of the buffer, number of bytes read, and the error.

```go
func main() {
	f, _ := os.Open("test.txt")         // 1
	defer f.Close()

	buf := make([]byte, 1024)           // 2
	n, err := f.Read(buf)               // 3

	str := string(buf)                  // 4
	fmt.Println(str, n, err)            // 5
}
```

### Buffered

The `bufio` package provides `NewReader`, which wraps an existing reader with an in-memory buffer. Buffered I/O reads from memory rather than making a system call on every operation, reducing overhead.

Use buffered I/O in the following circumstances:

| Use case | Example |
|:---|:---|
| Text input line by line | Wrap stdin or a file: `reader := bufio.NewReader(os.Stdin)` then `line, _ := reader.ReadString('\n')` |
| Parsing structured files (CSV, JSONL, logs) | Use buffering to efficiently read large files without loading them fully in memory. |
| From network sockets efficiently | `bufio.NewReader(conn)` minimizes syscalls when reading variable-sized messages. |
| Interactive CLI input | Use `ReadString('\n')` to get user input with editing or line buffering. |
| Scanning tokens or prefixes | Use `Peek()` or `ReadBytes()` to inspect part of the stream without consuming all data yet. |

The reader fills the buffer each time it reads data. This example reads text line by line:

1. Open the file. The file is a reader.
2. Create a buffered reader.
3. Read the file in an infinite `for` loop.
4. `ReadString` reads until it reaches the given delimiter, then returns a string and an error.
5. Check for EOF and return when you reach it.
6. Do something with the data during each loop.

```go
func main() {
	file, err := os.Open("source.txt")                  // 1
	if err != nil {
		log.Fatalln("Error opening file:", err)
	}
	defer file.Close()

	reader := bufio.NewReader(file)                     // 2

	for {                                               // 3
		line, err := reader.ReadString('\n')            // 4
		if err != nil {
			if err == io.EOF {                          // 5
				fmt.Print(line)
				break
			}
			log.Fatalf("Error reading line:", err)
		}
		fmt.Printf("%s", line)                          // 6
	}
}
```

By default, the buffer is 4,096 bytes (4KB). To change the size, use `bufio.NewReaderSize`. This example creates a 20-byte buffer and reads data one byte at a time:

1. Create a reader from a string.
2. Wrap the string reader with a buffered reader of size 20 bytes.
   {{< admonition "Internal minimum buffer size" note >}}
   The minimum buffer size in bytes is 16. If you create a buffer smaller than 16 bytes, Go silently rounds the buffer size up to 16.
   {{< /admonition >}}
3. Log the buffer size with `Size()`.
4. Loop through `stringReader` one byte at a time.
5. `ReadByte()` returns a single byte and an error.
6. If you reach the end of the file, return.
7. Log the byte read, how many bytes remain in the buffer (`Buffered()`), and the buffer size.

```go
func main() {
	data := "This is data that we will read one byte at a time."
	stringReader := strings.NewReader(data)                         // 1

	bufferedSize := 20
	br := bufio.NewReaderSize(stringReader, bufferedSize)           // 2

	fmt.Printf("Reader buffer size: %d\n\n", br.Size())             // 3

	for i := 0; i < len(data); i++ {                                // 4
		b, err := br.ReadByte()                                     // 5
		if err != nil {
			if err == io.EOF {                                      // 6
				fmt.Println("End of file")
				break
			}
			fmt.Printf("Error reading byte: %v\n", err)
			return
		}

		fmt.Printf("Read byte: %c (Buffered: %d/%d)\n", b, br.Buffered(), br.Size())    // 7
	}
}
```

This outputs the following:

```bash
Reader buffer size: 20

Read byte: T (Buffered: 19/20)
Read byte: h (Buffered: 18/20)
Read byte: i (Buffered: 17/20)
Read byte: s (Buffered: 16/20)
Read byte:   (Buffered: 15/20)
Read byte: i (Buffered: 14/20)
Read byte: s (Buffered: 13/20)
Read byte:   (Buffered: 12/20)
Read byte: d (Buffered: 11/20)
Read byte: a (Buffered: 10/20)
Read byte: t (Buffered: 9/20)
Read byte: a (Buffered: 8/20)
Read byte:   (Buffered: 7/20)
Read byte: t (Buffered: 6/20)
Read byte: h (Buffered: 5/20)
Read byte: a (Buffered: 4/20)
Read byte: t (Buffered: 3/20)
Read byte:   (Buffered: 2/20)
Read byte: w (Buffered: 1/20)
Read byte: e (Buffered: 0/20)
Read byte:   (Buffered: 19/20)
...
```

### ReadAll

`io.ReadAll` reads the entire contents of a reader into a byte slice without requiring you to pre-allocate the buffer:

1. Open the file. The file is a reader.
2. Pass the file handle to `ReadAll`. It returns a slice of bytes and an error.
3. Convert the slice of bytes to a string.
4. Output the file contents and the error.

```go
func main() {
	f, _ := os.Open("test.txt")     // 1
	defer f.Close()

	bytes, err := io.ReadAll(f)     // 2

	str := string(bytes)            // 3
	fmt.Println(str, err)           // 4
}
```

### As a parameter

Functions often take an `io.Reader` as a parameter and call `Read` on it internally.

`strings.NewReader` converts a string into a reader, letting you pass string data anywhere an `io.Reader` is expected:

1. Create a string variable.
2. Convert the string to a reader. `NewReader` takes a string and returns a reader.
3. Pass the reader to `ReadAll`. It calls `Read` on the reader and returns a slice of bytes and an error.
4. Convert the slice of bytes to a string.
5. Output the contents and the error.

```go
func main() {
	str := "Read this data."                // 1
	reader := strings.NewReader(str)        // 2

	bytes, err := io.ReadAll(reader)        // 3
	contents := string(bytes)               // 4
	fmt.Print(contents, err)                // 5
}
```

## io.Writer

A writer is a destination you can push bytes into. Data flows from memory through the writer into the output:

```
[]bytes -> Writer -> output destination
```

The `io.Writer` interface has one method:

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Go provides buffered and unbuffered writers. This table summarizes when to use each:

| Feature | Regular writer (`os.File`) | Buffered writer (`bufio.Writer`) |
|:---|:---:|:---:|
| Writes directly to OS | ✅ | ❌ |
| Batches small writes | ❌ | ✅ |
| Fewer system calls | ❌ | ✅ |
| Must call `Flush()` | ❌ | ✅ |
| Best for | Large single writes | Many small writes |

### Unbuffered

A file in Go is a writer because the [File interface](https://pkg.go.dev/io/fs#File) implements `Write`. This example writes an in-memory slice of bytes to a file:

1. Create a file and return a file handle. The file is a writer.
2. Create an in-memory slice of bytes.
3. `Write` writes the slice to the file and returns the number of bytes written and an error.
4. Output the number of bytes written and the error.

```go
func main() {
	f, _ := os.Create("output.txt")             // 1
	defer f.Close()

	data := []byte("Output from a stream.")     // 2

	n, err := f.Write(data)                     // 3
	fmt.Println(n, err)                         // 4
}
```

### Buffered

See [Copy from reader to writer](#copy-from-reader-to-writer).

### As a parameter

Functions often take an `io.Writer` as a parameter and call `Write` on it internally.

`fmt.Fprintf` is one example. It takes a writer and writes a formatted string to it. `bytes.Buffer` implements both `io.Reader` and `io.Writer`:

1. Create the `bytes.Buffer`. `buf` is a writer.
2. Write data as a formatted string to `buf`.
3. Extract the data from the buffer.

```go
func main() {
	var buf bytes.Buffer
	fmt.Fprintf(&buf, "Hello, %s.", "Charles")
	str := buf.String()
}
```

## Copy from reader to writer

`io.Copy` with a buffered writer is the most efficient way to read from a reader and write to a writer, avoiding expensive low-level methods:

1. Make an HTTP GET request.
2. Create a file to write the response to. The file handle is a writer.
3. Create a buffered writer. `NewWriter` wraps a writer and returns one with a 4KB buffer by default.
4. `io.Copy` reads the response body and writes it to the buffered writer in 4KB chunks until there is nothing left to read.
5. Flush the writer to disk. Because the buffered writer waits until its buffer is full, `Flush` ensures no bytes remain in the buffer after the copy completes. It writes to the underlying writer, which here is `file`.

```go
func main() {
	url := "https://www.example.com"
	r, err := http.Get(url)                 // 1
	if err != nil {
		log.Fatalln("Cannot get URL", err)
	}
	defer r.Body.Close()

	file, _ := os.Create("copy.html")       // 2
	defer file.Close()

	writer := bufio.NewWriter(file)         // 3
	io.Copy(writer, r.Body)                 // 4
	writer.Flush()                          // 5
}
```

## Reading files

Go provides a few options for reading files:

- `ReadFile`: Loads the entire file into memory and closes it automatically. Good for small to medium files.
- `os.Open` + `Read`: Reads into a buffer and lets you control how much data is read at a time. Good for large files.

This table compares use cases for each method:

| Use case | Recommended |
|:---|:---|
| Small file (config.json, < 1MB) | `os.ReadFile` |
| Large log file (1GB+) | `os.Open` + `Read` |
| Stream or pipe (stdin, socket) | `os.Open` or `bufio.Reader` |
| Test fixture or static HTML | `os.ReadFile` |
| Continuous data read | `os.Open` |

This table summarizes the features:

| Feature | `os.ReadFile` | `os.Open` + `Read` |
|:---|:---:|:---:|
| Reads all at once | ✅ | ❌ |
| Stream / partial read | ❌ | ✅ |
| Automatic close | ✅ | ❌ |
| Control over buffer | ❌ | ✅ |
| Best for small files | ✅ | ⚠️ |
| Best for large files | ❌ | ✅ |
| Memory use | High | Low |
| Simplicity | Very simple | More code, more control |

### ReadFile

`ReadFile` loads the entire file into memory and closes it automatically. Good for small to medium files:

1. `ReadFile` returns a slice of bytes and an error.
2. Convert the bytes into a string.
3. Do some work with the string data.

```go
func main() {
	bytes, err := os.ReadFile("source.txt")         // 1
	if err != nil {
		log.Println("Cannot read file: ", err)
	}
	str := string(bytes)                            // 2
	fmt.Println(str)                                // 3
}
```

### os.Open and Read

Opening and reading the file manually gives you more control over how much data you read. This example creates a buffer the size of the entire file:

1. `Open` returns a file handle in read-only mode. For other options, use `OpenFile`.
2. Always close the file.
3. Get the file size with `Stat` so you know how large to make the buffer.
4. Create a buffer the size of the file.
5. The file is a reader. Pass the buffer to `Read` to store its contents.
6. Log the number of bytes read from the file.
7. Do some work with the file.

```go
func main() {
	f, err := os.Open("source.txt")                     // 1
	if err != nil {
		log.Println("Cannot read file: ", err)
	}
	defer f.Close()                                     // 2

	stat, err := f.Stat()                               // 3
	if err != nil {
		log.Println("Cannot read file stats: ", err)
	}

	buf := make([]byte, stat.Size())                    // 4

	bytes, err := f.Read(buf)                           // 5
	if err != nil {
		log.Println("Cannot read buffer: ", err)
	}

	fmt.Printf("Read %d bytes from file\n", bytes)      // 6
	fmt.Println(string(buf))                            // 7
}
```

## Writing files

Go provides a few options for writing to files:

- `WriteFile`: Simplest method. Loads the entire content into memory and writes it at once. Good for small to medium files.
- `os.Open` and `Write`: Lets you perform multiple writes and gives you control over how you write (append, truncate, position). Good for streaming or large file writes.

This table compares use cases:

| Use case | Recommended |
|:---|:---|
| Small config file | `os.WriteFile` |
| Large file (stream or batch writes) | `os.OpenFile` + `Write` |
| Append logs continuously | `os.OpenFile(..., os.O_APPEND, ...)` |
| Write structured output in chunks | `os.OpenFile` + `Write` or `bufio.Writer` |
| Simple output to disk | `os.WriteFile` |

This table summarizes the features:

| Feature | `os.WriteFile` | `os.OpenFile` + `Write` | `os.OpenFile` + `bufio.Writer` |
|:---|:---:|:---:|:---:|
| Writes all at once | ✅ | ❌ | ❌ |
| Stream / chunk writing | ❌ | ✅ | ✅ |
| Buffering | ❌ | ❌ | ✅ |
| Simplicity | ✅ | ⚠️ | ⚠️ |
| Performance (small data) | ✅ | ✅ | ✅ |
| Performance (many small writes) | ❌ | ⚠️ | ✅ |
| Automatic close | ✅ | ❌ | ❌ |

### WriteFile

`WriteFile` loads all data into memory and writes to a file at once. It's the simplest method and best for small to medium files:

1. Data to write to the file.
2. `WriteFile` takes a file name, a slice of bytes, and Unix file permissions. `0644` gives the owner read and write permissions and other users read-only. If the file doesn't exist, `WriteFile` creates it. If it does, it clears the file and writes the new data without changing permissions.
3. Check for errors.

```go
func main() {
	data := []byte("WriteFile operation for small files!")  // 1

	err := os.WriteFile("writefile.txt", data, 0644)        // 2
	if err != nil {                                         // 3
		log.Println("Cannot write to file: ", err)
	}
}
```

### os.Create/OpenFile and Write

Use `os.Create` or `os.OpenFile` when you need multiple writes, control over file flags, or want to stream data rather than hold it all in memory at once:

1. Data to write to the file.
2. Create a destination file. `Create` creates a file with `0666` permissions if it doesn't exist. If it does exist, it clears the contents and preserves the permissions. Alternatively, use `OpenFile` with explicit flags: create if absent, open write-only, and truncate existing contents.
3. Always close the file.
4. Write the data to the file.
5. Log the number of bytes written.
6. Do some work with the data.

```go
func main() {
	data := []byte("WriteFile operation for streams or large files!")   // 1

	f, err := os.Create("create.txt")                                   // 2
    // f, err := os.OpenFile("output.txt", os.O_CREATE|os.O_WRONLY|os.O_TRUNC, 0644)
	if err != nil {
		log.Println("Cannot create file: ", err)
	}
	defer f.Close()                                                     // 3

	n, err := f.Write(data)                                             // 4
	if err != nil {
		log.Println("Cannot write to file: ", err)
	}
	fmt.Printf("Wrote %d bytes to file\n", n)                           // 5
	fmt.Println(string(data))                                           // 6
}
```

## Temporary files

A temporary file stores data for the duration of a program task and is deleted when the task completes or the data is persisted elsewhere. Temp files are created with `0600` permissions by default.

Use cases:

- Unit tests that need a temporary workspace
- Temporary file uploads or transformations
- Caching intermediate data
- Writing logs or output that doesn’t need to persist

This example creates and cleans up a temporary file:

1. Create a temporary directory. `os.TempDir()` returns the host system’s temp directory. The second argument is a name pattern where `*` is replaced with a random string.
2. Use `os.RemoveAll` to delete the temp directory and all its contents when the function returns.
3. `os.CreateTemp` creates a temporary file in `tmpdir`. The name follows the same `*` pattern.
4. `os.Remove` removes the temp file by name. This is redundant here since `os.RemoveAll` in step 2 handles cleanup, but demonstrates the standalone call.
5. Byte data to write to the temp file.
6. Write `data` to the temp file.
7. Log the data written and the temp file path.
8. `Close` releases the file handle and flushes any buffered data to disk.

```go
func main() {
	tmpdir, err := os.MkdirTemp(os.TempDir(), "tmpdir_*")                   // 1
	if err != nil {
		log.Println("Cannot create temp directory: ", tmpdir)
	}
	defer os.RemoveAll(tmpdir)                                              // 2

	tmpfile, err := os.CreateTemp(tmpdir, "tmpfile_*")                      // 3
	if err != nil {
		log.Println("Cannot create temp file: ", tmpfile)
	}
	defer os.Remove(tmpfile.Name())                                         // 4

	data := []byte("Random data for temporary file.")                       // 5
	_, err = tmpfile.Write(data)                                            // 6
	if err != nil {
		log.Println("Cannot write to temp file: ", tmpfile)
	}
	fmt.Printf("Wrote \"%s\" to %s\n", string(data), tmpfile.Name())        // 7

	err = tmpfile.Close()                                                   // 8
	if err != nil {
		log.Println("Cannot close temp file", err)
	}
}
```

### Temp files vs files

Temporary files and standard files share the same methods. The difference is how they are created, where they live, and how they are cleaned up:

| Use case | `os.CreateTemp` | `os.Create` / `os.Open` |
|:---|:---|:---|
| **Purpose** | Create a uniquely named temporary file | Create or open a specific file |
| **Location** | System temp directory by default, or a custom path | Anywhere you specify |
| **Name pattern** | Generates a random name from a prefix pattern | You specify the file name |
| **Cleanup** | Use `defer os.Remove()` or `RemoveAll()` | File persists until you delete it manually |
| **Security** | Atomic creation avoids race conditions | No automatic protection from name collisions |
| **Permissions** | Default `0600` (owner-only access) | Specify via `os.OpenFile` |
