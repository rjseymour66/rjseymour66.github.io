+++
title = 'Files and directories'
date = '2025-11-28T08:58:30-05:00'
weight = 20
draft = false
+++

## File and directory information

`os.Stat` gets information about a file or directory. It takes a file name string and returns a `FileInfo` type and an `error`. Here is the information available in `FileInfo`:

```go
type FileInfo interface {
	Name() string       // base name of the file
	Size() int64        // length in bytes for regular files; system-dependent for others
	Mode() FileMode     // file mode bits
	ModTime() time.Time // modification time
	IsDir() bool        // abbreviation for Mode().IsDir()
	Sys() any           // underlying data source (can return nil)
}
```
Here is a simple program that returns information about `test.txt`:

```go
func main() {
	info, err := os.Stat("test.txt")
	if err != nil {
		panic(err)
	}

	fmt.Printf("File name: %s\n", info.Name())
	fmt.Printf("File size: %d\n", info.Size())
	fmt.Printf("File permissions: %s\n", info.Mode())
	fmt.Printf("Last modified: %s\n", info.ModTime())
}
```

### Checking errors

When `os.Stat` returns an error, it is important to check the error to understand why you couldn't open the file. Use `errors.Is` with these errors from the `os` package:

```go
var (
	// ErrInvalid indicates an invalid argument.
	// Methods on File will return this error when the receiver is nil.
	ErrInvalid = fs.ErrInvalid // "invalid argument"

	ErrPermission = fs.ErrPermission // "permission denied"
	ErrExist      = fs.ErrExist      // "file already exists"
	ErrNotExist   = fs.ErrNotExist   // "file does not exist"
	ErrClosed     = fs.ErrClosed     // "file already closed"

	ErrNoDeadline       = errNoDeadline()       // "file type does not support deadline"
	ErrDeadlineExceeded = errDeadlineExceeded() // "i/o timeout"
)
```

For example:
1. Try to open a file that doesn't exist.
2. Check for the `ErrNotExist` error.

```go
func main() {
	info, err := os.Stat("example.txt")             // 1
	if err != nil {
		if errors.Is(err, os.ErrNotExist) {         // 2
			fmt.Println("File does not exist.")
			return
		} else {
			panic(err)
		}
	}
    // ...
}
```

## Linux file types

Linux has multiple file types. You can get details about a file type from the [`FileMode` object and its methods](https://pkg.go.dev/io/fs#FileMode), which is returned from `FileInfo.Mod()`.

Regular files
: Common data files containing text, images, or programs. The first character of the file listing is a dash (`-`). Check whether the file is a regular file with the `IsRegular` method on `FileMode`:

  ```go
  func main() {
	info, err := os.Stat("test.txt")
	if err != nil {
		if errors.Is(err, os.ErrNotExist) {
			log.Fatalln("File does not exist.")
		} else {
			panic(err)
		}
	}

	isReg := info.Mode().IsRegular()            // true
  }
  ```

Directories
: Hold other files and directories. The first character of the file listing is `d`. Check whether the file is a direcotry with the `IsDir` method on `FileMode`:
  ```go
  func main() {
	info, err := os.Stat("testdir")
	if err != nil {
		if errors.Is(err, os.ErrNotExist) {
			log.Fatalln("File does not exist.")
		} else {
			panic(err)
		}
	}

	isDir := info.Mode().IsDir()
	fmt.Println(isDir)
  }
  ```

Symbolic links
: A symbolic link is a pointer to another file. The first character of the file listing is `l`. Check whether a file is a symlink by checking `Lstat` and `info.Mode()&os.ModeSymLink` is equal to `0`. Non-zero means the file is not a symlink:

  ```go
  func main() {
	info, err := os.Lstat("symlink.file")
	if err != nil {
		// handle error
	}

	if info.Mode()&os.ModeSymlink != 0 {
		fmt.Println("This is a symlink")
	} else {
		fmt.Println("Not a symlink")
	}
  }
  ```

Named pipes (FIFOs)
: A named pipe is a mechanism for inter-process communication (IPC). The first character of the file listing is `p`. Check whether a file is a FIFO with `os.ModeNamedPipe`:

  ```go
  func main() {
	info, err := os.Stat("pipe")
	if err != nil {
		// handle error
	}

	if info.Mode()&os.ModeNamedPipe != 0 {
		fmt.Println("This is a named pipe")
	} else {
		fmt.Println("Not a named pipe")
	}
  }
  ```

Character devices
: Character devices provide unbuffered, direct access to hardware devices. The first character of the file listing is `c`. Check whether a file is a character device with `os.ModeCharDevice`:

  ```go
  func main() {
	info, err := os.Stat("/dev/tty")
	if err != nil {
		// handle error
	}

	if info.Mode()&os.ModeCharDevice != 0 {
		fmt.Println("This is a char device")
	} else {
		fmt.Println("Not a char device")
	}
  }
  ```

Block devices
: Block devices provide buffered access to hardware devices. The first character of the file listing is `b`. Go does not provide a direct method for checking block devices.

Sockets
: A socket is an endpoint for communication. The first character of the file listing is `s`. Check whether a file is a socket with `os.ModeSocket`:
  
  ```go
  func main() {
	info, err := os.Stat("/tmp/mysock")
	if err != nil {
		// handle error
	}

	if info.Mode()&os.ModeSocket != 0 {
		fmt.Println("This is a socket")
	} else {
		fmt.Println("Not a socket")
	}
  }
  ```

## Permissions


Linux permissions are commonly represented in the human-readable octal notation. It is a sum of the read (4), write (2), and execute (1) bits. Set the file permissions to the octal value `761`:

```bash
chmod 761 test.txt
ll test.txt 
-rwxrw---x 1 username username 89 Nov 28 09:04 test.txt*
```

Retrieve the file permissions with the `Perm` method. They are returned in the same format as the bash `ls -l` command, but you can convert them to their octal representation with `Sprintf` and the `%o` formatting verb:
1. Get the file information.
2. Retrieve the permissions. They are returned in `-rwxrw---x` format.
3. Convert the permissions to octal.
4. Print the permissions. This outputs `761`.

```go
func main() {
	info, err := os.Stat("test.txt")                        // 1
	if err != nil {
		// handle error
	}

	permissions := info.Mode().Perm()                       // 2
	permissionsString := fmt.Sprintf("%o", permissions)     // 3
	fmt.Printf("Permissions: %s\n", permissionsString)      // 4
}
```

## File paths

A file path in Go is a string representation of a file or directory location in the filesystem. Directory names are separated by a path separator, which is a forward slash in Linux and macOS (`home/file/path`), and a backslash in Windows (`C:\file\path`). Go's `path/filepath` package is platform-independent, so you can use the same code for all OSs.

### Joining file paths

Create a file path with the `Join` method. It accepts a variable number of `string` arguments and concatenates them using the correct path separator.

When you define the strings, you do not have to worry about whether there is a trailing or leading slash in your string. The `Join` function will build the file path correctly.

Here, the strings all beggin and end with different slash formats and the path output is correct. In production, you should use a consistent style:

```go
func main() {

	dir := "/home/username/filepath/"
	subdir := "level1/level2"
	file := "document.txt"

	fullPath := filepath.Join(dir, subdir, file) 
	fmt.Println(fullPath)   // /home/username/filepath/level1/level2/document.txt
}
```

### Cleaning file paths

In some cases, the file path portions that you want to concatenate might contain redundant separators or refrerences to the current directory (`.`) or parent directory (`..`). Use `Clean` to resolve these issues and return the shortest path name equivalent to the provided input.

{{< admonition "" note >}}
`Clean` does not resolve file paths, it only transforms them into more readable paths.
{{< /admonition >}}

For example, the following `uncleanPath` contains multiple separators and a reference to the parent directory. `Clean` removes the redundant separators and then "follows" the parent path to remove `/user/` from the output:

```go
func main() {
	uncleanPath := "/home/user///../documents/file.txt"
	cleanPath := filepath.Clean(uncleanPath)            // /home/documents/file.txt
}
```

### Splitting file paths

You can separate a path into directory and file parts with `Split`. A few things to consider:
- If the `path` argument ends with a path separator, the returned file value is empty.
- If there is no path separator in the `path` argument, the returned directory value is empty.

```go
func main() {
	path := "/home/username/filepath/level1/level2/test.txt"
	dir, file := filepath.Split(path)
	fmt.Println(dir)             // /home/username/filepath/level1/level2/
	fmt.Println(file)            // test.txt
}
```

### Relative paths

Get the relative directory path between two filesystem locations with `Rel`. This method accepts a base path and a target path and returns the relative path from the end of the base path to the last directory. For example, if the target path ends with a file name, it returns the path to its parent directory. If the target path ends in a directory, it returns the path to that directory.

The target path must contain the base path, or you will get an error:

```go
func main() {
	basePath := "/home/username/"
	targetPath := "/home/username/filepath/level1/level2/test.txt"
	relDir, err := filepath.Rel(basePath, filepath.Dir(targetPath))
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(relDir)         // filepath/level1/level2
}
```

### Absolute path

Get the absolute path to a file or directory with `Dir`. If the given path ends with a slash, `Dir` returns the given path with no trailing slash. Under the hood, `Dir` calls `Clean` on the path, so it returns a trailing slash only when the returned value is the root directory:

```go
func main() {
	file := "/home/username/filepath/level1/level2/test.txt"
	dir := "/home/username/filepath/level1/level2/"
	fmt.Println(filepath.Dir(file))         // /home/username/filepath/level1/level2
	fmt.Println(filepath.Dir(dir))          // /home/username/filepath/level1/level2
}
```

### Last element in path

`Base` returns the last element in a given path:

```go
func main() {
	dir := "/home/username/"
	file := "/home/username/filepath/level1/level2/test.txt"
	fmt.Println(filepath.Base(dir))         // username
	fmt.Println(filepath.Base(file))        // test.txt
}
```

## Traversing directories

{{< admonition "`fs.Walk` vs `fs.WalkDir`" note >}}
`fs.WalkDir` was introduced in Go 1.16, and is more lightweight than `fs.Walk` because of the callbacks used for each function:\

```go
type WalkFunc func(path string, info fs.FileInfo, err error) error  // fs.Walk
type WalkDirFunc func(path string, d fs.DirEntry, err error) error  // fs.WalkDir
```

`WalkFunc` uses a `FileInfo` struct, which makes a system call on every file. `WalkDirFunc` only makes the system call if you call `d.Info()` on the `DirEntry` struct.

Use `WalkFunc` if you require a `FileInfo` struct.

{{< /admonition >}}

You can traverse directories and perform actions on the directories and files within them with the `WalkDir` method. `WalkDir` takes a `path` and an `fs.WalkDirFunc`, and it returns an `error`. `fs.WalkDirFunc` is a callback that is invoked on each file or directory in the given `path`.

`path`
: Accepts both relative and absolute paths. If `path` ends in file, the callback is invoked once. If `path` ends in a directory, it is invoked recursively. You can normalize `path` with `Clean` before passing to `WalkDir`.

`fs.WalkDirFunc`
: `fs.WalkDirFunc` is a callback that you call on each file or directory in the path given to `WalkDir`. It has the following signature:
  
  ```go
  type WalkDirFunc func(path string, d DirEntry, err error) error
  ```
  - `path`: The file or directory currently being visited.
  - `d`: An `fs.DirEntry` struct that gives you access to methods that help you determine whether you want to operate on the current `path`:
    - `d.Name()`: Returns the name of the current `path`. 
    - `d.IsDir()`: Boolean, returns whether `path` is a directory.
    - `d.Type()`: Returns the file type. For details, see [Linux file types](#linux-file-types).
    - `d.Info()`: [Returns `FileInfo`](#file-and-directory-information) for the current path, identical to `os.Stat`. 
  - `err`: Handle an error returned by the callback.
  When you call `WalkDir`, you do not pass arguments to `WalkDirFunc`, only the parameters. `WalkDir` passes the arguments to `WalkDirFunc` as it traverses the file system, so define it as an anonymous functionj

  Here is an example of how to get information from the `fs.DirEntry` struct:
  1. Get the name.
  2. Check if the file is a directory.
  3. Check if the file is a regular file.
  4. Get the `FileInfo` for the file.
  5. These four lines show how you can retrieve the file details from the `FileInfo` struct.

  ```go
  func main() {
	path := "/home/ryanseymour/filepath/"
	cleanPath := filepath.Clean(path)

	err := filepath.WalkDir(cleanPath, func(path string, d fs.DirEntry, err error) error {
		if err != nil {
			return err
		}

		fmt.Println(d.Name())                           // 1
		fmt.Println(d.IsDir())                          // 2
		fmt.Println(d.Type().IsRegular())               // 3

		info, err := d.Info()                           // 4
		if err != nil {
			return err
		}
		fmt.Printf("File name: %s\n", info.Name())      // 5
		fmt.Printf("File size: %d\n", info.Size())
		fmt.Printf("File permissions: %s\n", info.Mode())
		fmt.Printf("Last modified: %s\n", info.ModTime())

		return nil
	})

	if err != nil {
		panic(err)
	}
  }
  ```

### SkipDir and SkipAll

`SkipDir` and `SkipAll` are error types that you can return from the callback in `WalkDir`. They determine how `WalkDir` proceeds when it visits files or directories that meet a specific condition:

| Return value | Effect                                             |
| ------------ | -------------------------------------------------- |
| `fs.SkipDir` | Skip this directory, continue walking other paths. |
| `fs.SkipAll` | Stop the entire walk immediately.                  |

This example demonstrates both return values. It skips the `.git` directory, and it stops traversing the file system when it encounters a regular file named `stop-here.txt`:

1. Define the path you want to start the directory traversal.
2. Check for an unexpected error.
3. Call `SkipDir` if the file is a directory and the directory name is `.git`. This continues traversing the file system.
4. If the file is a regular file named `stop-here.txt`, call `SkipAll`. This will stop the file system traversal and return to `main`.

```go
func main() {
	traversePath := "testdir"                                           // 1 
	err := filepath.WalkDir(traversePath, func(path string, d fs.DirEntry, err error) error {
		if err != nil {                                                 // 2
			return err
		}

		if d.IsDir() && d.Name() == ".git" {                            // 3
			return fs.SkipDir
		}

		if d.Type().IsRegular() && d.Name() == "stop-here.txt" {        // 4
			fmt.Printf("Stopped at %s\n", d.Name())
			return fs.SkipAll
		}
		fmt.Println(path)
		return nil
	})

	if err != nil {
		fmt.Errorf("Error walking %s: %v", traversePath, err)
	}
}
```

## Symbolic links

A symblolic link is a pointer to another file. Create a symbolic link when you want convenient access to a file in another directory. To delete a symlink, [remove the file](#removing-files).
 
This example creates a symbolic link with `os.Symlink`. `Symlink` takes a source path (original file) and symlink path (pointer to the original file):
1. Define the path for the original file.
2. Define the path where you want to create the symbolic link.
3. Call `Symlink`.
4. Check for errors.

```go
func main() {
	sourcePath := "testdir/one/two/three/original.txt"                      // 1
	symLinkPath := "testdir/one/symlink.txt"                                // 2

	err := os.Symlink(sourcePath, symLinkPath)                              // 3
	if err != nil {                                                         // 4
		fmt.Printf("Error creating symlink: %v\n", err)
		return
	}
}
```

Verify the program with `tree`:

```go
testdir
└── one
    ├── symlink.txt -> testdir/one/two/three/original.txt                   // symlink
    └── two
        ├── fourth.txt
        └── three
            ├── four
            ├── original.txt                                                // original file
            └── stop-here.txt
```

### Resolving symlinks

You can find which file a symlink points to with `os.Readlink`, which returns the original file that the given symlink points to:

```go
func main() {
	orig, err := os.Readlink("testdir/one/symlink.txt")
	if err != nil {
		fmt.Errorf("Error: %v\n", err)
	}
	fmt.Println(orig)       // testdir/one/two/three/original.txt
}
```

The following example uses `fs.Walk` to verify that all symlinks resolve to a valid target:
1. Check for an unexpected error.
2. Check if the file is a symlink.
3. If it is a symlink, get the symlink's target with `os.Readlink`.
4. If `os.Readlink` doesn't return an error, get information about the symlink target with `os.Stat`.
5. If `os.Stat` returns an error, use `errors.Is` to check whether the error is `ErrNotExist`.
6. Handle other errors.

```go
func main() {
	dir := "testdir"
	err := filepath.Walk(dir, func(path string, info fs.FileInfo, err error) error {
		if err != nil {                                                                     // 1
			fmt.Fprintf(os.Stdout, "Error accessing path %s: %v\n", path, err)
			return nil
		}

		if info.Mode()&os.ModeSymlink != 0 {                                                // 2
			target, err := os.Readlink(path)                                                // 3
			if err != nil {
				fmt.Fprintf(os.Stdout, "Error reading symlink %s: %v\n", path, err)
			} else {
				_, err := os.Stat(target)                                                   // 4
				if err != nil {
					if errors.Is(err, os.ErrNotExist) {                                     // 5
						fmt.Fprintf(os.Stdout, "Broken symlink found: %s -> %s\n", path, target)
					} else {                                                                // 6
						fmt.Fprintf(os.Stderr, "Error checking symlink target %s: %v\n", target, err)
					}
				}
			}
		}
		return nil
	})

	if err != nil {
		fmt.Fprintf(os.Stderr, "Error walking directory %s: %v\n", dir, err)
	}
}
```

## File system operations

### Removing files

Remove a file with `os.Remove`. It accepts a file path and returns an `error`:

1. Define the file path.
2. Remove the file with `os.Remove`.

```go
func main() {
	junkFile := "testdir/one/two/three/four/junk.file"      // 1

	err := os.Remove(junkFile)                              // 2
	if err != nil {
		fmt.Printf("Error removing the file: %v\n", err)
		return
	}
	fmt.Printf("File removed: %s\n", junkFile)
}
```

### Calculating directory size

This function calculates the size of all files in the directory:
1. Declare an accumulator variable to hold the size in bytes.
2. You need `FileInfo`, so use the `Walk` function.
3. Check if the file is a directory.
4. If the file is not a directory, add its size to the `size` variable.
5. If `Walk` returned an error, return `0` and the error.
6. Return the `size` and `nil` for the `error`.

```go
func calculateDirSize(path string) (int64, error) {
	var size int64                                                  // 1

	err := filepath.Walk(path,                      
		func(path string, info fs.FileInfo, err error) error {      // 2
			if err != nil {
				return err
			}
			if !info.IsDir() {                                      // 3
				size += info.Size()                                 // 4
			}
			return nil
		})
	if err != nil {                                                 // 5
		return 0, err
	}

	return size, nil                                                // 6
}
```

### Finding duplicate files

To find duplicate files, you need to find the MD5 checksum of the file, which is a digital fingerprint of the file and its contents. This function returns a hexadecimal string representation of the file hash:

1. Open the file.
2. Check for errors.
3. Make sure the file closes when the function exits.
4. Create an MD5 hash object. This hasher is a Writer. As you write bytes to it, it updates the hash state.
5. Use `io.Copy` to read from the file and write to the hasher. This feeds data into the MD5 algorithm. `io.Copy` is efficient because it streams the file---it does not read the entire file into memory.
6. Return the has as a lowercase hexadecimal string. `hash.Sum(b []byte)` appends the hash digest to the slice `b`. If you pass `nil`, then you get only the hash output.

```go
func computeFileHash(filePath string) (string, error) {
	file, err := os.Open(filePath)                          // 1
	if err != nil {                                         // 2
		return "", err
	}
	defer file.Close()                                      // 3

	hash := md5.New()                                       // 4
	if _, err := io.Copy(hash, file); err != nil {          // 5
		return "", err
	}
	return fmt.Sprintf("%x", hash.Sum(nil)), nil            // 6
}
```

This function returns a map of duplicate files, where the hash is the key and the file path is the value. It uses `computeHash` to get the MD5 file hash:

{{< admonition "Symbolic links" tip >}}
Symbolic links might cause this function to behave incorrectly, so you might want to skip symbolic links when detecting duplicate files.
{{< /admonition >}}

1. Declare the `duplicates` map with a `string` key and a slice of strings value.
2. You need `FileInfo`, so use the `Walk` function.
3. Check for unexpected errors.
4. Check if the file is not a directory.
5. If the file is not a directory, pass the `path` to `computeFileHash` to get the MD5 hash.
6. Record the file path under its hash. If the hash key already has a value, the file path is appended to the slice of strings.
7. Return the map and `nil` error.

```go
func findDuplicateFiles(rootDir string) (map[string][]string, error) {
	duplicates := make(map[string][]string)                                 // 1
	err := filepath.Walk(rootDir, 
		func(path string, info fs.FileInfo, err error) error {              // 2
			if err != nil {                                                 // 3
				return err
			}

			if !info.IsDir() {                                              // 4
				hash, err := computeFileHash(path)                          // 5
				if err != nil {
					return err
				}

				duplicates[hash] = append(duplicates[hash], path)           // 6
			}
			return nil
		})
	return duplicates, err                                                  // 7
}
```

### Memory-mapped files


```go
func main() {
	filePath := "example.txt"
	file, err := os.OpenFile(filePath, os.O_RDWR|os.O_CREATE, 0644)
	if err != nil {
		fmt.Printf("Failed to open file: %v\n", err)
		return
	}
	defer file.Close()

	fileInfo, err := file.Stat()
	if err != nil {
		fmt.Printf("Failed to get file info: %v\n", err)
		return
	}
	fileSize := fileInfo.Size()

	data, err := syscall.Mmap(int(file.Fd()), 0, int(fileSize), syscall.PROT_READ|syscall.PROT_WRITE, syscall.MAP_SHARED)

	if err != nil {
		fmt.Printf("Failed to mmap file: %v\n", err)
		return
	}

	defer syscall.Munmap(data)

    func main() {
	filePath := "example.txt"
	file, err := os.OpenFile(filePath, os.O_RDWR|os.O_CREATE, 0644)
	if err != nil {
		fmt.Printf("Failed to open file: %v\n", err)
		return
	}
	defer file.Close()

	fileInfo, err := file.Stat()
	if err != nil {
		fmt.Printf("Failed to get file info: %v\n", err)
		return
	}
	fileSize := fileInfo.Size()

	data, err := syscall.Mmap(int(file.Fd()), 0, int(fileSize), syscall.PROT_READ|syscall.PROT_WRITE, syscall.MAP_SHARED)

	if err != nil {
		fmt.Printf("Failed to mmap file: %v\n", err)
		return
	}

	defer syscall.Munmap(data)

	fmt.Printf("Initial content: %s\n", string(data))
	newContent := []byte("Hello, mmap!")
	copy(data, newContent)
	fmt.Println("Content updated successfully.")
}
```