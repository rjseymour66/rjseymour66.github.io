---
title: "Reading, writing, files"
weight: 50
description: >
  Working with files. Reading and writing data, too.
---

## File operations

#### Get file name

```go
name := fileName.Name()
```
#### File extension
```go
ext := filepath.Ext(file)
```
#### Delete a file
```go
if err := os.Remove; err != nil {
	// handle err
}
```

#### File metadata

Use [fs.FileInfo](https://pkg.go.dev/io/fs#FileInfo) to examine file metadata, such as the name, length in bytes, if it is a directory, etc. To return the `FileInfo` file attributes for a file, use `os.Stat(filename)`:
```go
info, err := os.Stat(fileName)
if err != nil {
	// handle error
}
```

#### Open a file

To open a file for reading (`O_RDONLY`), use the `Open` function:

```go 
in, err := os.Open(filename)
if err != nil {
	return nil
}
```

When you need to do more than read a file, use  `os.OpenFile()`. `os.OpenFile()` accepts flags (`O_APPEND`) so you can perform more actions:
```go
f, err = os.OpenFile(*logFile, os.O_APPEND|os.O_CREATE|os.O_RDWR, 0644)
if err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
}
// always defer the close
defer f.Close()
```

## Read a file

Read data from a file with the `os` package. `ReadFile` reads the contents of a file and returns a `nil` error:
```go
fileContents, err := os.ReadFile("filename")
	if err != nil {
		if errors.Is(err, os.ErrNotExist) {
			return nil
		}
		return err
	}
if len(fileContents) == 0 {
    return nil
}
```
The previous example returns `nil` if the file does not exist. If your program creates a file after this code runs--for example, with `os.WriteFile`--then the program returns `nil` and continues.

### Scanner for lines and words

The Scanner type accepts an `io.Reader` and reads data that is delimited by spaces or new lines.

By default, it reads lines, but you can configure it to read words:

```go
scanner := bufio.NewScanner(r)
// scan words
scanner.Split(bufio.ScanWords)
```
Use the `.Scan()` function with a default scanner to read a single line. Check the `.Err()` value, then return the text with the `.Text()` function:

```go
s.Scan()

if err := s.Err(); err != nil {
    return "", err
}

if len(s.Text()) == 0 {
    return "", fmt.Errorf("Input cannot be empty")
}

return s.Text(), nil
```

Use `.Scan()` in a for loop to read lines or tokens, depending on the `.Split()` configuration:
```go
for s.Scan() {
    // if non-EOF error
    if err := s.Err(); err != nil {
        return "", err
    }
    file = s.Text()

    if len(s.Text()) == 0 {
        return "", fmt.Errorf("File cannot be blank")
    }
}
```

To find the number of bytes in each scanned token:
```go
// scan words
scanner.Split(bufio.ScanWords)

byteLength := 0
for scanner.Scan() {
    byteLength += len(scanner.Bytes())    
}
```
### Buffered reading

When you open a file with `os.Open`, the computer uses the default buffer size for the `os.File` type. This default size is os-specific, which does not provide control over your input/output (I/O) operations.

Reading a file with a buffered reader lets you change the file handle's buffer size, which gives you control over how many system calls the OS makes during I/O. The Go `bufio` package provides the [`NewReaderSize` function](https://pkg.go.dev/bufio#NewReaderSize) that returns a `bufio.Reader` whose buffer has at least the specified size. Buffer size is important because it controls how much memory the OS consumes for the operation.

The following example creates a reader with a 1MB buffer to read from a file:

```go
// open file
f, err := os.Open(...)

bReader := bufio.NewReaderSize(f, 1024 * 1024)
if err := json.NewDecoder(bReader).Decode(&typeName); err != nil {
	return nil, err
}
```

> You do not have to close a `bufio.Reader`

### CSV data

Create a `.NewReader()` the same way that you create a `.NewScanner()` and read with the following methods:
- `.Read()` returns a `[]string` that represents a row.
- `.ReadAll()` returns a `[][]string`, where each slice is a row in the CSV file. 

Below is an example that reads an entire CSV file and tries to convert the values to `float64`:
```go
func csv2float(r io.Reader, column int) ([]float64, error) {
	// Create the CSV Reader used to read in data from CSV files
	cr := csv.NewReader(r)
	// adjusting column arg for 0-based index
	column--
	// Read in all CSV data
	allData, err := cr.ReadAll()
	if err != nil {
		return nil, fmt.Errorf("Cannot read data from file: %w", err)
	}

	var data []float64
	/*
		convert [][]string to [][]float64
	*/

	// loop through all records
	for i, row := range allData {
		// skip the first row that contains the column headers
		if i == 0 {
			continue
		}
		// Checking number of cols in CSV file to verify flag value
		if len(row) <= column {
			// file does not have that many columns
			return nil,
				fmt.Errorf("%w: File has only %d columns", ErrInvalidColumn, len(row))
		}
		// Try to convert data read into a float number
		v, err := strconv.ParseFloat(row[column], 64)
		if err != nil {
			return nil, fmt.Errorf("%w: %s", ErrNotNumber, err)
		}

		data = append(data, v)
	}
	return data, nil
}
```
## Write to a file

Write data to a file with the `os` package. `WriteFile` writes to an existing file or creates one, if necessary:

```go
os.WriteFile(name string, data []byte, perm FileMode) error

```
> **Linux permissions**: Set Linux file permissions for the file owner, group, and all other users (`owner-group-others`). The permission options are read, write, and execute. Use octal values to set permssions:  
  read: 4  
  write: 2  
  execute: 1  

When you assign permissions in an programming language, you have to tell the program that you are using the octal base. You do this by beginning the number with a `0`. So, `0644` permissions means that the file owner has read and write permissions, and the group and all other users have read permissions.

### io.Writer

Commonly named `w` or `out`. Examples of `io.Writer`:
- os.Stdout
- bytes.Buffer (implements `io.Writer` (and `io.Reader`) as a pointer receiver, so use `&`)
- files (type os.File implements `io.Writer`)
- gzip.Writer

> Use a file or `os.Stdout` in the program, and `bytes.Buffer` when testing.

### Buffered writing

When you open a file with `os.Open`, the computer uses the default buffer size for the `os.File` type. This default size is os-specific, which does not provide control over your input/output (I/O) operations. Writing to a file with a buffered writer lets you change how much data you write with each call, which means you do not have perform I/O intensive operations like writing line-by-line. 

The Go `bufio` package provides the [`NewWriterSize` function](https://pkg.go.dev/bufio#NewWriterSize) that returns a `bufio.Writer` whose buffer has at least the specified size. You write data with the `bufio.Write` method. This method only writes data when the buffer is full--this means that you must call the `.Flush()` method to write the final chunk of data:

```go
// open file
f, err := os.Open(...)

bWriter := bufio.NewWriterSize(f, 1024 * 1024)
for _, data := contents {
  _, err = buffedWriter.Write(data)
  if err != nil { ... }
}

if err := bWriter.Flush; err != nil {
	// handle err
}
```

### Archive files

Compress files to an `io.Writer` with the [`compress/gzip` package](https://pkg.go.dev/compress/gzip@go1.20.2). You can create a writer with `NewWriter`, and write compressed data to the `io.Writer` argument. Assign the `Name` value to the name of the source file:

```go
out, err := os.OpenFile(targetPath, os.O_RDWR|os.O_CREATE, 0755)
if err != nil {
    return err
}
defer out.Close()

...

// create new zip writer
zw := gzip.NewWriter(out)
zw.Name = filepath.Base(path)

// copy contents
if _, err = io.Copy(zw, in); err != nil {
    return err
}
```

One method of compressing a file is to use the `io.Copy` method, passing the gzip writer as the destination writer:
```go
if _, err = io.Copy(zw, in); err != nil {
    return err
}
```

After you write all data, you must close the gzip writer and the writer that you passed as the argument to the `NewWriter` constructor. Handle the error when you close the gzip writer--don't defer it--because you want to know if the compression fails:

```go
if err := zw.Close(); err != nil {
	return err
}

if err := out.Close(); err != nil { 
	// handle err 
}
```

### tabWriter

`tabwriter.Writer` writes tabulated data with formatted columns with consistent widths using tabs (`\t`).

https://pkg.go.dev/text/tabwriter#pkg-overview


## Buffers and bytes

Many functions return `[]byte`, so you might have to fill a buffer with data to return.

The `bytes` package contains two types: `Buffer` and `Reader`. The `Buffer` returns a variable size buffer to read and write data. It provides `Write*` methods for `strings`, `runes`, `bytes`, etc.:

```go
func byteStuff() []bytes {
    // compose the page using a buffer of bytes to write to a file
    var buffer bytes.Buffer

    // write html to bytes buffer
    buffer.WriteString("The first string")
    buffer.Write([]byte{'T', 'h', 'e', 's', 'e', 'c', 'o', 'n', 'd', 's', 't', 'r', 'i', 'n', 'g'})
    buffer.WriteString("The last string")

    // return []bytes
    return buffer.Bytes()
}
```

## Temp files

If you need to create a temp file:
```go
temp, err := os.CreateTemp("", "pattern*.extension")
if err != nil {
    // handle error
}
defer temp.Close()
defer os.Remove(temp.name())
```
- First parameter is the directory that you want to create the temporary file in. If it is left blank, it uses the `/tmp` directory.
- The second parameter defines the file name. Use a `*` character to tell the program to generate a random number to make the name unique.

Always close and remove temp files with `defer`, unless you are creating a test helper. For test helpers, see `t.Cleanup()`.

## Directory paths

The `filepath` library creates cross-platform filepaths--they compile correctly for each supported OS.

Get the relative directory path between a root and target path:
```go
relDir, err := filepath.Rel(root, filepath.Dir(path))
if err ...
```

Extract the last element in a filepath--generally, the file--with the filepath.Base() function:
```go
filename := filepath.Base("/path/to/home.html")
// filename == home.html
```
Return the absolute path of a file:
```go
	absPath := "/home/username/dev/go-projects/walker/main.go"
	fmt.Println(filepath.Dir(absPath)) // /home/username/dev/go-projects/walker
```

Return the relative path between two paths:
```go
absPath := "/home/username/dev/go-projects/walker/main.go"
relPath, err := filepath.Rel("/home", absPath)
if err != nil {
	fmt.Fprintln(os.Stderr, err)
}
fmt.Println(relPath) //username/dev/go-projects/walker/main.go
```

Return the base of the path. This is generally a file name:

```go
fullPath := "/home/username/dev/go-projects/walker/main.go"
base := filepath.Base(fullPath)
fmt.Println(base) // main.go
```

Join multiple paths into a single path:

```go
start := "/home"
middle := "username/dev/go-projects"
end := "/walker/main.go"
fmt.Println(filepath.Join(start, middle, end)) // /home/username/dev/go-projects/walker/main.go
```
`os.MkdirAll` creates a directory tree from its `path` argument. It only creates directories that do not exist. If the path exists, it does nothing.