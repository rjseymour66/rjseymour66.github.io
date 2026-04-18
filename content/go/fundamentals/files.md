+++
title = 'Files'
date = '2025-09-06T00:01:29-04:00'
weight = 60
draft = false
+++


## Reading files

Go's `os` package provides several ways to read a file. Choose based on file size and whether you need to process data all at once or as a stream:

| Method | Description | Use case |
|:---|:---|:---|
| `os.ReadFile` | Reads the entire file into memory at once | Small, bounded files |
| `os.Open` + `bufio.Scanner` | Reads tokenized input one token at a time | Line-by-line text, log files |
| `os.Open` + `bufio.Reader` | Reads raw bytes or strings with full EOF control | Custom protocols, binary formats |

### ReadFile

`os.ReadFile` reads the entire file into memory and returns a byte slice. It's the simplest option, but it has tradeoffs:

- It preallocates a byte slice before reading. Large files consume a lot of memory.
- You can't read the data as a stream or process it in chunks.
- It blocks until the entire file is read. Slow disk reads block the goroutine.
- A malicious input pointing to a large file can cause a denial-of-service.

Use `ReadFile` for small, bounded files:

```go
func main() {
	data, err := os.ReadFile("structured.log")
	if err != nil {
		log.Fatal(err)
	}
	log.Println(string(data))
}
```

### Open

`os.Open` opens a file for reading and returns a file descriptor. Use it with a `Scanner` or `Reader` to process the file as a stream:

```go
func lineRead(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	// Scanner or Reader to process
}
```

### Scanner

A [Scanner](https://pkg.go.dev/bufio#Scanner) reads tokenized input. A token is a unit of data bounded by a separator. By default, the separator is a newline. The scanner keeps only one token in memory at a time, making it safe for large files.

{{< admonition "Default token size" warning >}}
A scanner has a default token size of 64KB. If you need to read larger tokens, increase the buffer size with `scanner.Buffer()`.
{{< /admonition >}}

This example reads a file line by line:

1. Open the file.
2. Defer the close immediately after opening.
3. Create a scanner and pass the file descriptor.
4. `Scan()` advances to the next token. It returns `false` at EOF or on error.
5. `Text()` returns the current token as a string.
6. Check for non-EOF errors after the loop.

```go
func lineRead(filename string) {
	file, err := os.Open(filename)           // 1
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()                       // 2

	scanner := bufio.NewScanner(file)        // 3

	for scanner.Scan() {                     // 4
		line := scanner.Text()               // 5
		fmt.Println(line)
	}

	if err := scanner.Err(); err != nil {    // 6
		log.Fatal(err)
	}
}
```

Change the delimiter with `Split`. Use a [SplitFunc](https://pkg.go.dev/bufio#SplitFunc) for a custom delimiter, or one of the built-in functions to scan by byte, word, or rune. This example scans by word:

```go
scanner := bufio.NewScanner(file)
scanner.Split(bufio.ScanWords)

for scanner.Scan() {
	fmt.Println(scanner.Text())
}
```

### Reader

A [Reader](https://pkg.go.dev/bufio#Reader) gives you granular control over reads. Unlike a `Scanner`, it can return raw bytes and expose `io.EOF` directly. Use it when parsing custom protocols or binary formats.

This example reads a file line by line:

1. Open the file.
2. Defer the close immediately after opening.
3. Create a `Reader`. `NewReader` accepts a file descriptor and returns a `Reader` with an underlying buffer.
4. Loop until EOF.
5. `ReadString` reads until the delimiter and returns everything up to and including it.
6. If the error is `io.EOF`, print the final chunk and stop.

```go
func lineRead(filename string) {
	file, err := os.Open(filename)              // 1
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()                          // 2

	reader := bufio.NewReader(file)             // 3
	for {                                       // 4
		line, err := reader.ReadString('\n')    // 5
		if err == io.EOF {                      // 6
			fmt.Print(line)
			break
		}
		if err != nil {
			log.Fatal(err)
		}
		fmt.Print(line)
	}
}
```

#### File metadata

`os.Open` returns a file descriptor. Call `Stat` on it to read file metadata:

1. Open the file.
2. Defer the close.
3. Call `Stat` to retrieve the `FileInfo`.

```go
func main() {
	file, err := os.Open("structured.log")  // 1
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()                      // 2

	info, err := file.Stat()                // 3
	if err != nil {
		log.Fatal(err)
	}

	log.Printf("\nFile name:\t%s\nFile mode:\t%v\nFile size:\t%d\nDirectory:\t%v\n",
		info.Name(), info.Mode(), info.Size(), info.IsDir())
}
```

## Writing files

Go's `os` package provides several ways to write to a file, depending on whether you need simple writes, streaming, or control over file flags:

| Method | Description | Use case |
|:---|:---|:---|
| `os.Create` | Creates or truncates a file and returns a handle | Simple writes with data already in memory |
| `io.Copy` | Streams data from a `Reader` to a `Writer` using a buffer | Copying files, streaming large data |
| `os.OpenFile` | Opens a file with explicit flags and permissions | Append mode, custom permissions, read-only |

### Create

`os.Create` creates or truncates a file and returns a file handle. If the file exists, it opens it. If it doesn't, it creates it. Write strings with `WriteString` or raw bytes with `Write`:

```go
func main() {
	file, err := os.Create("test.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	file.WriteString("test")
}
```

### io.Copy (buffered writes)

`os.Create` requires data already in memory. To stream data into a write, use `io.Copy`.

`io.Copy(dest, src)` uses a 32KB buffer to read from the source `Reader` and write to the destination `Writer` until EOF. It returns the number of bytes written and an error:

1. Open the source file.
2. Defer the close.
3. Create or open the destination file.
4. Defer the close.
5. Copy from source to destination. Discard the byte count but handle the error.

```go
func bufferedWrites(destination, source string) error {
	src, err := os.Open(source)             // 1
	if err != nil {
		return err
	}
	defer src.Close()                       // 2

	dest, err := os.Create(destination)     // 3
	if err != nil {
		return err
	}
	defer dest.Close()                      // 4

	_, err = io.Copy(dest, src)             // 5
	if err != nil {
		return err
	}
	return nil
}
```

### OpenFile

`os.OpenFile` opens a file with explicit flags and permissions. Combine flags with the bitwise OR (`|`) to control how the file opens.

This example appends to an existing file. The flags open the file for writing only, create it if it doesn't exist, and append to the end if it does:

```go
func main() {
	file, err := os.OpenFile("test.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	if _, err := file.WriteString("test two!"); err != nil {
		log.Fatal(err)
	}
}
```