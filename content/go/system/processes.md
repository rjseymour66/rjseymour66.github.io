+++
title = 'Processes and pipes'
date = '2025-12-12T23:41:03-05:00'
weight = 60
draft = false
+++


The `os/exec` package contains the types and methods to run external processes.

## Execute commands

This trivial example runs executes the Bash command `sleep 2`, which makes the terminal sleep for two seconds. You can manage the process with a context with a timeout and channels:

1. `exec.Command` takes a `string` command and a variable number of `string` arguments so you can pass multiple flags or options.
2. Define a custom timeout. This timeout is longer than the `sleep` argument so there is enough time for `sleep` to complete.
3. Create a cancellable context with a timeout deadline. This returns a context and a cancel function.
4. Defer the cancel function so that it is called either when the function exits or the context expires.
5. Run the command with `Start`. The command is run asynchronously, so it does not block.
6. Create a `done` channel of type error to track when the command completes.
7. In a goroutine, run the command `Wait` method. This method waits for the command to complete, and then returns an error. Run it in a goroutine because it will block execution.
8. The `select` statement waits for events on channels.
   1. Wait for the context to fire the `Done` method. `Done` is a channel that closes when the timeout expires. When `Done` sends a value, use `cmd.Process.Kill` to stop the process. Handle any errors.
   2. If the command does not complete before the timeout expires, receive an error from the `done` channel.

```go
func main() {
	cmd := exec.Command("sleep", "2")                                       // 1
	timeout := 3 * time.Second                                              // 2

	ctx, cancel := context.WithTimeout(context.Background(), timeout)       // 3
	defer cancel()                                                          // 4

	if err := cmd.Start(); err != nil {                                     // 5
		panic(err)
	}

	done := make(chan error, 1)                                             // 6
	go func() {                                                             // 7
		done <- cmd.Wait()
	}()

	select {                                                                // 8
	case <-ctx.Done():                                                      // 8.1
		if err := cmd.Process.Kill(); err != nil {
			panic("failed to kill process: " + err.Error())
		}
		fmt.Println("Process killed as timeout reached")

	case err := <-done:                                                     // 8.2
		if err != nil {
			panic("process finished with error: " + err.Error())
		} else {
			fmt.Println("Process finished successfully")
		}
	}
}
```


## IPC and pipes

Inter-process communication (IPC) allows efficient data transfer between system processes. A pipe is an in-memory conduit for transporting data between two or more processes. Pipes use the _consumer-producer_ model: one process produces data and funnels it into a pipe, and the consumer process reads data from that stream.

Pipes work well when processes specialize in specific tasks. These processes can communicate directly without intermediate storage. Pipes are common in these scenarios:
- Command-line utilities
- Data streaming
- Inter-process data exchange

Here is a basic example of a pipe:

```bash
cat file.txt | grep "passwd"
```

### Pipes vs channels

Pipes and channels seem similar but have different use cases. Pipes require more setup, but they are useful in these scenarios:
- Communication with different processes across different programming languages
- An application that has multiple executables that need to communicate
- You work in a Unix environment

Channels are a feature in Go, so use them in these scenarios:
- Concurrent applications that need to communicate across goroutines
- Complex concurrency patterns, such as fan-in, fan-out, worker pools, etc.

### Anonymous pipes

An anonymous pipe is an unnamed, in-memory byte stream that connects a producer to a consumer (Writer to Reader). It is called "anonymous" because it does not have a name in the filesystem, and it only exists while the processes are running.

Here is an example of an anonymous pipe in bash:
```bash
echo "Hello world!" | grep "Hello"
```

The following Go code executes the equivalent of the preceding bash command:
1. Defines the `echo` command. `exec.Command` takes a string command and a variable number of string arguments so you can pass multiple flags or options.
2. `Output()` starts `echoCmd` and waits for it to finish, capturing its stdout in a `[]byte`.
3. Defines the `grep` command. Here, `grep` checks its input for the string `Hello`.
4. Assigns `echoOutput` to `grep`'s stdin. `echoOutput` is a `[]byte`, so you cast it as a string. `Cmd.Stdin` is a Reader, so you wrap it in a `strings.Reader` before assigning it as the stdin for `grepCmd`.
5. Use `Output()` to get the stdout of `grepCmd`.
6. Print `grepCmd` to the console.

```go
func main() {
	echoCmd := exec.Command("echo", "Hello world!")

	echoOutput, err := echoCmd.Output()
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error running echoCmd: %v\n", err)
		return
	}

	grepCmd := exec.Command("grep", "Hello")
	grepCmd.Stdin = strings.NewReader(string(echoOutput))

	grepOutput, err := grepCmd.Output()
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error running grepCmd: %v\n", err)
		return
	}

	fmt.Printf("Output of grep: %s", grepOutput)
}
```

## Named pipes

Named pipes enable local streaming communication between independent programs on your computer without using a network. They can be used between any processes (they don't have to be live), and they persist in the filesystem. Think of a named pipe as a shared mailbox that lives at a known address (location in the file system) with the following properties:
- One program sends messages in
- Another program picks up messages
- Messages come out in the same order they went in (FIFO)
- Nothing is saved to disk
- Messages wait until a program picks them up


This example creates a named pipe. Create a named pipe with `Mkfifo()`:


1. The path to the named pipe.
2. Check if the named pipe exists with a helper function.
3. If the pipe does not exist, create a pipe named `mailbox` with `Mkfifo()`. This creates a named pipe at the given path with world-readable and -writeable permissions with `0666`.
4. Open `mailbox` for reading and writing. This prevents blocking---pipes block if they are opened for reading with no writers, or writing with no readers.
5. Defer closing the mailbox until the function returns.
6. Set up a WaitGroup for concurrency.
7. Tell the WaitGroup to wait for two processes.
8. In a goroutine, run a helper function `ReadTask` that reads from a pipe. This helper uses a scanner to read data. Defer a call to the WaitGroup's `Done` function.
9. In another goroutine, send 10 "tasks" to the pipe. Each task line ends with a newline (`/n`) so the scanner in `ReadTask` can read the message. Defer a call to the WaitGroup's `Done` function.
10. Wait for the goroutines to finish. `Wait` executes when all tasks are written, and all tasks are read.

```go
func main() {
	mailboxPath := "/tmp/task_mailbox"                                          // 1
	if !namedPipeExists(mailboxPath) {                                          // 2
		fmt.Println("The mailbox does not exist")
		fmt.Println("Creating the task mailbox...")
		if err := unix.Mkfifo(mailboxPath, 0666); err != nil {                  // 3
			fmt.Println("Error setting up the task mailbox", err)
			return
		}
	}

	mailbox, err := os.OpenFile(mailboxPath, os.O_RDWR, os.ModeNamedPipe)       // 4
	if err != nil {
		fmt.Println("Error opening named pipe:", err)
	}
	defer mailbox.Close()                                                       // 5

	wg := &sync.WaitGroup{}                                                     // 6
	wg.Add(2)                                                                   // 7

	go func() {                                                                 // 8
		defer wg.Done()
		ReadTask(mailbox)
	}()

	go func() {                                                                 // 9
		defer wg.Done()

		i := 0
		for i < 10 {
			SendTask(mailbox, fmt.Sprintf("Task %d\n", i))
			i++
		}
		SendTask(mailbox, "EOD\n")
		fmt.Println("All tasks sent.")
	}()

	wg.Wait()                                                                   // 10
}
```

The helper function `SendTask` writes data to a string:
1. You don't need to count the number of bytes written, so ignore it.
2. Return an error if there is an issue writing the data to the pipe.
3. Return `nil` if there are no errors.
```go
func SendTask(pipe *os.File, data string) error {
	_, err := pipe.WriteString(data)                                // 1
	if err != nil {                                                 // 2
		return fmt.Errorf("error writing to named pipe: %v", err)
	}
	return nil                                                      // 3
}
```

`ReadTask` uses a scanner to read from the pipe until a message containing only "EOD" is sent:
1. Create a scanner that reads from the pipe.
2. Scan a line of text. By default, scanners read until they reach a newline delimiter.
3. If `task` is "EOD", break from the scanning loop.
4. Check for non-EOF errors.

```go
func ReadTask(pipe *os.File) error {
	fmt.Println("Reading tasks from the mailbox...")

	scanner := bufio.NewScanner(pipe)                       // 1
	for scanner.Scan() {                                    // 2
		task := scanner.Text()
		fmt.Printf("Processing task: %s\n", task)

		if task == "EOD" {                                  // 3
			break
		}
	}

	if err := scanner.Err(); err != nil {                   // 4
		return fmt.Errorf("error reading tasks from the mailbox: %v", err)
	}
	fmt.Println("All tasks processed.")
	return nil
}
```

Here is the helper function that checks whether the pipe exists. It returns a Boolean---if the pipe exists, it returns `true`, otherwise, it returns `false`:
1. Get file information with `os.Stat`. We don't need the file information, so ignore the `FileInfo` return value.
2. If there was no error, the pipe exists. Return `true.`
3. If there is an error, use `errors.Is` to check whether it is an `ErrNotExist` error, then return `false`.
4. If there is some other error, return `false`.

```go
func namedPipeExists(pipePath string) bool {
	_, err := os.Stat(pipePath)                         // 1
	if err == nil {                                     // 2
		return true
	}

	if errors.Is(err, os.ErrNotExist) {                 // 3
		return false
	}

	fmt.Println("Error checking named pipe:", err)
	return false                                        // 4
}
```

### Log processing tool

This tool is a live log filter built on a named pipe (FIFO). It simulates a logger that writes to a pipe and a function that filters for error logs:

```
log writer > named pipe > filterLogs func > stdout
```

#### Log filterer

The `filterLogs` function uses a scanner to read log entries and check for error lines. It takes a reader and a writer, and it writes any error logs to the Writer:

```go
func filterLogs(reader io.Reader, writer io.Writer) {
	scanner := bufio.NewScanner(reader)
	for scanner.Scan() {
		logEntry := scanner.Text()
		if strings.Contains(logEntry, "ERROR") {
			writer.Write([]byte(logEntry + "\n"))
		}
	}
}
```

#### Create read-only pipe

`main` defines the pipe, opens it for writing, then defines a goroutine that opens it for reading. These processes are in the same program, but they could be completely separate. This demonstrates a decoupled producer and consumer, where the producer behaves like a never-ending file stream that blocks until the consumer reads data from the pipe:

1. Defines the filesystem location of the pipe.
2. If there is already a pipe that exists in that location, it deletes the pipe.
3. Create a named pipe. `0600` permissions grant the user executing the program with read/writer permissions.
4. Cleans up the pipe when the program exits.
5. Opens a FIFO read-only pipe that blocks until there is a writer. If the pipe exists (it does), then `os.O_CREATE` is ignored.
6. Close the pipe when `main` exits.

```go
func main() {
	pipePath := "/tmp/my_log_pipe"                          // 1
	if err := os.RemoveAll(pipePath); err != nil {          // 2
		panic(err)
	}
	if err := unix.Mkfifo(pipePath, 0600); err != nil {     // 3
		panic(err)
	}
	defer os.RemoveAll(pipePath)                            // 4

	pipeFile, err := os.OpenFile(                           // 5
        pipePath, 
        os.O_RDONLY|os.O_CREATE, 
        os.ModeNamedPipe,
    )
	if err != nil {
		panic(err)
	}
	defer pipeFile.Close()                                  // 6
```

#### Create writer that reads from pipe

Start the writer goroutine. The goroutine simulates a separate process that writes logs, such as an `slog`:
1. Opens the pipe for write only (`os.O_WRONLY`). This unblocks the `OpenFile` call in the `main` goroutine.
2. Close the pipe when `main` exits.
3. Simulates the logger. It writes an INFO and and ERROR log every second. Scanners depend on newline delimiters, so each log message ends with a `\n` character.
4. Filters the logs for ERROR messages.


```go
    // main continued...
	go func() {
		writer, err := os.OpenFile( 
			pipePath,
			os.O_WRONLY,
			os.ModeNamedPipe,
		)
		if err != nil {
			panic(err)
		}
		defer writer.Close()

		for {
			writer.WriteString("INFO: All systems operational\n")
			writer.WriteString("ERROR: An error occurred\n")
			time.Sleep(1 * time.Second)
		}
	}()

	filterLogs(pipeFile, os.Stdout)
}
```