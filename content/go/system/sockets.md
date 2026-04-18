+++
title = 'UNIX Sockets'
date = '2025-12-27T12:30:11-05:00'
weight = 70
draft = false
+++

UNIX sockets---also called UNIX domain sockets---are a local alternative to TCP/IP sockets for IPC. They provide a way for processes to communicate with each other on the same machine quickly and efficiently. UNIX sockets can be stream- or datagram-oriented, and they are represented as files or directories. They have the following features:

- **Efficiency**: You can transfer data between processes without a network. Even calling localhost requires that you use the TCP/IP stack.
- **Filesystem** namespace: They are file system paths---they are easy to locate and persist until you remove them.
- **Security**: Manage access with standard filesystem permissions.


## Creating a socket

When you create a UNIX domain socket, Go creates a socket file descriptor in the OS and binds the socket it to the given file system path. 

1. Create the socket path. This file system location acts like an address. It is created when the server starts listening with `net.Listen`.
2. UNIX domain sockets leave a file behind when it stops. This removes any socket file in the location. Check the error with `errors.Is` to verify the error. If there is an error and the error is not because there is a socket file in the location, return the error.
3. `Listen` tells the OS to create a UNIX domain socket at the given path and start listening for connections. Pass `unix` to create a UNIX socket that accepts only local connections.
4. Close the socket when `main` exits.
5. Create a signal channel for a graceful shutdown.
6. Listen for interrupt and termination signals only.
7. In a separate goroutine, handle the signals:
   1. Wait for a signal on the `signals` channel. This blocks until a signal is received.
   2. Close the channel to stop receiving connections.
   3. Remove the socket file.
   4. Exit cleanly.
8. Accept connections in an infinite loop:
   1. Accept connections. This blocks until a client connects.
   2. Check for errors, but continue accepting connections if there is an error.
9. Handle connections with the helper function.


```go
func main() {
	socketPath := "/tmp/example.sock"                           // 1

	if err := os.Remove(socketPath); err != nil && !errors.Is(err, os.ErrNotExist) {    // 2
		log.Printf("Error removing socket file: %v", err)
		return
	}

	listener, err := net.Listen("unix", socketPath)             // 3
	if err != nil {
		log.Printf("Error listening: %v", err)
		return
	}
	defer listener.Close()                                      // 4
	fmt.Println("Listening on", socketPath)

	signals := make(chan os.Signal, 1)                          // 5
	signal.Notify(signals, syscall.SIGINT, syscall.SIGTERM)     // 6

	go func() {                                                 // 7
		<-signals                                               // 7.1
		fmt.Println("Received termination signal. Shutting down gracefully...")
		listener.Close()                                        // 7.2
		os.Remove(socketPath)                                   // 7.3
		os.Exit(0)                                              // 7.4
	}()

	for {                                                       // 8
		conn, err := listener.Accept()                          // 8.1
		if err != nil {                                         // 8.2
			log.Printf("Error accepting connection: %v", err)
			continue
		}

		go handleConnection(conn)                               // 9
	}
}
```

A function that handles the connection:
1. Close the connection when the function exits.
2. Create a 1KB buffer.
3. Read up to 1KB bytes and store it in the buffer. `Read` blocks until the client sends data.
4. Handle errors.
5. Write the client message to the console. `buffer[:n]` returns only what was written to the buffer. For example, if 10 bytes were written to the buffer, it would return only those 10 bytes, not the entire 1024 bytes available in the buffer.
6. Create a response for the client.
7. Write the response back to the connection.
8. Handle errors.

```go
func handleConnection(conn net.Conn) {
	defer conn.Close()                                                  // 1
	buffer := make([]byte, 1024)                                        // 2
	n, err := conn.Read(buffer)                                         // 3
	if err != nil {                                                     // 4
		log.Printf("Error reading from connection: %v", err)
		return
	}

	fmt.Println("Received:", string(buffer[:n]))                        // 5

	response := []byte("Message received successfully\n")               // 6
	_, err = conn.Write(response)                                       // 7

	if err != nil {                                                     // 8
		log.Printf("Error writing response to connection: %v", err)
		return
	}
}
```

### Inspecting the socket

The bash command `lsof` helps you inspect open files, include UNIX sockets:
- `-U`: Limits the file types to UNIX domain sockets
- `-a`: Acts like a conditional. It says "show file descriptors that are UNIX socket files AND match the following path":

```bash
lsof -Ua /tmp/example.sock 
#... warning messages
COMMAND       PID        USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
files-dir 1331190 ryanseymour    3u  unix 0xffff99840f382640      0t0 3344845 /tmp/example.sock type=STREAM
```

## Creating a client

This client connects to a local UNIX domain socket, sends a message, then accepts a server response:

1. `Dial` establishes a connection to the given UNIX socket path.
2. Check for errors.
3. Close the connection when the function finishes.
4. Sends the message to the server. This casts the message as a slice of bytes.
5. Handle errors.
6. Create a buffer for the server response. Here, the buffer is 1KB.
7. `Read` reads the server response and writes it to the buffer.
8. Handle errors.
9. Print the server response stored in the buffer.

```go
func main() {
	conn, err := net.Dial("unix", "/tmp/example.sock")          // 1
	if err != nil {                                             // 2
		fmt.Println("Error dialing:", err)
		return
	}
	defer conn.Close()                                          // 3

	_, err = conn.Write([]byte("Hello UNIX socket!\n"))         // 4
	if err != nil {                                             // 5
		fmt.Println("Error writing to socket:", err)
		return
	}

	buffer := make([]byte, 1024)                                // 6
	n, err := conn.Read(buffer)                                 // 7
	if err != nil {                                             // 8
		fmt.Println("Error reading from socket:", err)
		return
	}

	fmt.Println("Server response:", string(buffer[:n]))         // 9
}
```
