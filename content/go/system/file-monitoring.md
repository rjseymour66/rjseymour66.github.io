+++
title = 'File Monitoring'
date = '2025-12-07T10:20:34-05:00'
weight = 50
draft = false
+++

The easiest way to watch for changes to files is with the [`fsnotify` package](https://pkg.go.dev/github.com/fsnotify/fsnotify), a cross-platform package that wathces for file system events. To install, run this command in your terminal:

```bash
go get github.com/fsnotify/fsnotify
```
## fsnotify

This program watches a directory for changes to the file system. It uses a signal channel to listen for interrupts from the terminal:

1. Define the path that you want to watch.
2. Create a new `Watcher`. A Watcher watches for file system events on a channel of type `Event`.
3. Check for errors.
4. Close the Events channel right before the method returns.
5. `Add` registers the directory with the Watcher. The Watcher now monitors the path for changes.
6. Check for errors.
7. In a goroutine, run an inifinite loop with a `select` loop that watches for events on the Wather.
8. When `watcher.Events` sends a filesystem event, it prints it to the screen.
9. When the Watcher gets an error, it prints it to the screen.
10. Set up a signals channel.
11. Watch for interrupt signals from the terminal.
12. Block until there is a value sent from `signalCh`.

```go
func main() {
	watchPath := "./testdir"                                // 1

	watcher, err := fsnotify.NewWatcher()                   // 2
	if err != nil {                                         // 3
		log.Fatal("Error creating watcher:", err)
	}
	defer watcher.Close()                                   // 4

	err = watcher.Add(watchPath)                            // 5
	if err != nil {                                         // 6
		log.Fatal("Error adding watch:", err)
	}

	go func() {                                             // 7
		for {
			select {
			case event := <-watcher.Events:                 // 8
				fmt.Printf("Event: %s\n", event.Name)
			case err := <-watcher.Errors:                   // 9
				log.Println("Error:", err)
			}
		}
	}()

	signalCh := make(chan os.Signal, 1)                     // 10
	signal.Notify(signalCh, os.Interrupt, syscall.SIGINT)   // 11

	<-signalCh                                              // 12
	fmt.Println("Received SIGINT. Exiting...")
}
```

## File rotation

File rotation helps you efficiently manage and organize files so they do not consume too many resources. It is helpful in the following scenarios:

System logs
: File rotation makes sure that your OS and application log files do not become too large so you can preserve historical data.

Backup files
: Helps maintain recent and historical copies of important data.

Compliance logs
: Retains and organizes records for compliance and auditing purposes.

Application-specific data
: Some apps generate data files, such as transaction logs or user-generated content.

Web server logs
: Web servers track who visits their websites with logs. Rotate these logs to help with analysis and security monitoring.
Sensor data and IoT devices: These devices continually gather data, so you need to manage the resources consumed by the data files.

This example rotates a log file when it reaches 10MB. There should be an existing log file, or you can update this code to create one.

### Step 1: Constants and globals

First, define the constants and globals:
1. Path to the log file you are watching and rotating.
2. File size limit that you want to set on your log file.
3. Global variable for the log file when the program writes to it.

```go
const (
	logFilePath = "./testdir/logfile"   // 1
	maxFileSize = 1024 * 10 * 10        // 2
)

var logFile *os.File                    // 3
```

### Step 2: Set up watcher

In `main`, set up a watcher to monitor your `logFilePath`. The watcher has an `Events` channel that emits watcher events, like writes, creates, deletes, etc:
1. Create the watcher.
2. Close the watcher when `main` returns.
3. Add `logFilePath` to the watcher.

```go
func main() {
	watcher, err := fsnotify.NewWatcher()                   // 1
	if err != nil {
		log.Fatal("Error creating watcher:", err)
	}
	defer watcher.Close()                                   // 2

	err = watcher.Add(logFilePath)                          // 3
	if err != nil {
		log.Fatal("Error adding file to watcher:", err)
	}
```

### Step 3: Watch for file system events

Next, you need to watch the file for a write event. If there is a write event, you need logic that will rotate the file when it reaches the `maxFileSize`: 

1. Set up the mutex to prevent multiple concurrent writes to the log file.
2. Watch for file system events in a goroutine.
3. Create an inifinite `for` loop so you can read events as long as necessary.
4. Begin a `select` statement to watch the Watcher's `Events` channel for an event.
5. The first `select` case checks what kind of event the Watcher is emitting.
   1. Comma-ok checks whether the channel is opened or closed, and it returns if it is closed. `ok` is false when you receive the zero value for the channel type, which means the channel is closed.
   2. A bitmask check to see if there was a write operation on the file. `event.Op` is an integer that encodes one or more flags, and `fsnotify.Write` is a constant with one bit set. When you do a bitwise AND(`&`) operation between the two, it compares them bit-by-bit and produces a new value that contains only the bits that are set in both. So, if the write bit is set in `event.Op` and you bitwise AND it with `fsnotify.Write`, then it will be equal to `fsnotify.Write`.
   3. Get the file info so you can check its size.
   4. Get the file size.
   5. If `fileSize` is greater than or equal to the global `maxFileSize`, then use a mutex to lock the file, rotate the file, then unlock the file.
6. The second `select` case handles any errors on the channel and checks whether it is open or closed with the comma-ok idiom.

```go
    // main continued...
	var mu sync.mutex                                                           // 1

	go func() {                                                                 // 2
		for {                                                                   // 3
			select {                                                            // 4
			case event, ok := <-watcher.Events:                                 // 5
				if !ok {                                                        // 5.1
					return
				}
				if event.Op&fsnotify.Write == fsnotify.Write {                  // 5.2
					fi, err := os.Stat(logFilePath)                             // 5.3
					if err != nil {
						fmt.Println("Error getting file info:", err)
						continue
					}
					fileSize := fi.Size()                                       // 5.4
					if fileSize >= maxFileSize {                                // 5.5
						mu.Lock()
						rotateLogFile()
						mu.Unlock()
					}
				}
			case err, ok := <-watcher.Errors:                                   // 6
				if !ok {
					return
				}
				fmt.Println("Error watching file:", err)
			}
		}
	}()
```

### Step 4: Set up signal channels

Set up a channel to watch for signals from the terminal:
1. Set up a signals channel.
2. Watch for interrupt signals from the terminal.
3. Block until there is a value sent from `signalCh`.

```go
    // main continued ...
	signalCh := make(chan os.Signal, 1)                     // 1
	signal.Notify(signalCh, os.Interrupt, syscall.SIGINT)   // 2

	<-signalCh                                              // 3
	fmt.Println("Received SIGINT. Exiting...")
}
```
### Step 5: Rotate file function

`rotateLogFile` closes the log file, renames it, then creates a new log file. It uses helper functions to close the current log file and create a new file:

1. Close the log file if it is open.
2. Create a timestamp.
3. Create a unique file name with the timestamp.
4. Rename the `logFilePath` file (the file with the watcher) with the new timestamped filename. This is where the "log rotation" occurs.
5. Create a new log file and check for errors.

```go
func rotateLogFile() {
	err := closeLogFile()                                               // 1
	if err != nil {
		fmt.Println("Error closing log file:", err)
		return
	}

	timestamp := time.Now().Format("20060102150405")                    // 2
	newLogFilePath := fmt.Sprintf("your_log_file_%s.log", timestamp)    // 3

	err = os.Rename(logFilePath, newLogFilePath)                        // 4
	if err != nil {
		fmt.Println("Error renaming log file:", err)
		return
	}

	err = createLogFile()                                               // 5
	if err != nil {
		fmt.Println("Error creating new log file:", err)
		return
	}
}
```

### Step 6: Helper methods

The `closeLogFile` function checks whether there is a file assigned to the global `logFile` variable. If there is, then it closes the file. Otherwise, it returns an error:

```go
func closeLogFile() error {
	if logFile != nil {
		return logFile.Close()
	}
	return nil
}
```

The `createLogFile` function creates a new file and sets the logger to write to it:
1. Create a new file.
2. Set the output location with `SetOutput`.

```go
func createLogFile() error {
	logFile, err := os.Create("your_log_file.log")      // 1
	if err != nil {
		return err
	}
	log.SetOutput(logFile)                              // 2
	return nil
}
```