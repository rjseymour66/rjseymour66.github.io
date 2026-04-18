+++
title = 'TCP'
date = '2025-09-06T22:59:20-04:00'
weight = 10
draft = false
+++

Transmission Control Protocol (TCP) provides built-in handshaking, error detection, and reconnection features.

TCP uses a reliablity method called Go-Back-N to ensure there is no data loss in a connection. If a packet is lost, the sender rolls back and retransmits from the lost packet onward. This means that even received packets are retransmitted. 

## TCP server

TCP ensures a reliable and ordered delivery of birectional data with messag acknowledgements and sequences of data packets. A simple TCP server has the following parts:
- Listen for incoming connections.
- Accept the connection.
- Read and optionally write data to the connection.

### Netcat TCP server

To test TCP programs, you need a TCP server. Netcat is a simple command line utility that accepts simple test messages on the specified port and writes them to the console.

Run this command in a terminal to start a TCP server that listens continuously on port 1902:
- `-6`: Listen to IPv6
- `-l`: Listen
- `-k`: Keep listening
- `-u`: UDP protocol


```bash
nc -lk 1902
```

### Simple server

A socket is a combination of the protocol, IP address or hostname, and port number. The following simple TCP server reads from an infinite loop and handles each connection in a separate goroutine.

1. Establish the connection:
   1. `net.Listen` returns a generic network listener for stream-oriented protocols. It accepts two arguments: the protocol, and an address to listen on in _`host:port`_ format. If there is no hostname or IP address provided, it will listen on any interface for IPv4 or IPv6. If you provide the host, it listens only for IPv4. If you set the port to `0`, then a random port is chosen and you have to retrieve it with `net.Addr`.
   2. Handle any errors.
   3. Close the listener.
2. Listen for connections:
   1. Listen for connections in an infinite `for` loop.
   2. `Accept` is a method on the `Listener` interface. It blocks until a new connection arrives, and then returns a `Conn` struct that represents the next connection.
   3. Handle any errors for `Accept`.
3. Handle the connections. This logic could be extracted into a function and called in a goroutine, such as `handleConnection(c Conn)`.
   1. Handle each connection in a separate goroutine so the main thread can continue accepting connections. This IIFL accepts a `Conn` object named `c`. This ensures that each goroutine operates on a different connection variable. Passing `conn` directly to the closure would overwrite the connection in each goroutine.
   2. Close the connection when the goroutine exits.
   3. Create a buffer to store connection data.
   4. `Read` reads data from the connection into `buf`.
   5.  Handle errors for `Read`.
   6.  Log the contents of `buf` to the console. It is stored as a slice of bytes, so make sure you caste it as a string.
   7.  `Write` writes a slice of bytes to the connection. This sends a trivial response back to the server. If you wanted to log the contents of the buffer, you would replace this line with `Write(buf[:n])`.
   8.  Pass to the IIFL the `conn` value returned from `Accept`.
   

```go
func main() {
	listener, err := net.Listen("tcp", "localhost:9000") 	// 1.1
	if err != nil { 										// 1.2
		log.Fatal(err)
	}
	defer listener.Close() 									// 1.3

	for { 													// 2.1
		conn, err := listener.Accept() 						// 2.2
		if err != nil { 									// 2.3
			log.Fatal(err)
		}

		go func(c net.Conn) { 								// 3.1
			defer c.Close() 								// 3.2
			buf := make([]byte, 1024) 						// 3.3
			_, err := c.Read(buf) 							// 3.4
			if err != nil { 								// 3.5
				log.Fatal(err)
			}
			log.Print(string(buf)) 							// 3.6
			c.Write([]byte("Hello from TCP server\n")) 		// 3.7
		}(conn) 											// 3.8
	}
}
```


### Production server

Here is a more complicated TCP server that uses channels to handle connections. The code is explained in detail by [Matthew Slipper in a blog post](https://www.matthewslipper.com/2019/12/21/production-tcp-servers-in-go.html). Here are some more examples:
- [Linode TCP and UDP Client and Server in Go](https://www.linode.com/docs/guides/developing-udp-and-tcp-clients-and-servers-in-go/)
- [go-tcp-proxy](https://github.com/jpillora/go-tcp-proxy/blob/master/proxy.go)
- [Build a Blazing-Fast TCP Server in Go: A Practical Guide](https://dev.to/jones_charles_ad50858dbc0/build-a-blazing-fast-tcp-server-in-go-a-practical-guide-29d)

```go
const MaxLineLenBytes = 1024
const ReadWriteTimeout = time.Minute

func main() {
	lis, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatalf("failed to start listener: %v", err)
	}

	for {
		conn, err := lis.Accept()
		if err != nil {
			log.Printf("failed to accept conn: %v", err)
			continue
		}

		go handleConn(conn)
	}
}

func handleConn(conn net.Conn) {
	log.Printf("accepted connection from %s", conn.RemoteAddr())

	defer func() {
		_ = conn.Close()
		log.Printf("closed connection from %s", conn.RemoteAddr())
	}()
	done := make(chan struct{})

	// time out one minute from now if no
	// data is received
	_ = conn.SetReadDeadline(time.Now().Add(ReadWriteTimeout))

	go func() {
		// limit the maximum line length (in bytes)
		lim := &io.LimitedReader{
			R: conn,
			N: MaxLineLenBytes,
		}
		scan := bufio.NewScanner(lim)
		for scan.Scan() {
			input := scan.Text()
			output := strings.ToUpper(input)
			if _, err := conn.Write([]byte(output + "\n")); err != nil {
				log.Printf("failed to write output: %v", err)
				return
			}
			log.Printf("wrote response: %s", output)
			// reset the number of bytes remaining in the LimitReader
			lim.N = MaxLineLenBytes
			// reset the read deadline
			_ = conn.SetReadDeadline(time.Now().Add(ReadWriteTimeout))
		}

		done <- struct{}{}
	}()

	<-done
}
```

## TCP client

A TCP client is much simpler than a TCP server. All you have to do is `Dial` the server, write to the connection, and close the connection:
1. Open the connection to the server with the `Dial` function. `Dial` returns a generic `net.Conn` interface for packet-oriented connections. It is a Reader and a Writer.
2. Handle any errors.
3. Close the connection.
4. Write to the connection with `Write`.

```go
func main() {
	conn, err := net.Dial("tcp", ":9000") 		// 1
	if err != nil { 							// 2
		log.Fatal(err)
	}
	defer conn.Close() 							// 3
	conn.Write([]byte("Hello TCP client")) 		// 4
}
```

### UDP server

UDP is connectionless---the client sends data packets to the server, and the server sends data packets to the client. There is no acknowledgement that the packets are received on either end. The following example is a simple UDP server:
1. Listen for UDP connections on port 9001. `ListenPacket` returns a `net.PacketConn` interface for packet-oriented connections. This is not a Reader or a Writer. Rather, it has `ReadFrom` and `WriteTo` methods to read and write to the connection
2. Handle any errors.
3. Close the connection when `main` finishes.
4. Create a 1KB buffer for data you read from the connection.
5. Create an infinite loop that listens for packets on the connection.
6. `ReadFrom` blocks until it reads a packet from the connection. It returns the number of bytes read, the return IP address on the packet, and any error. 
7. Handle any errors.
8. (Optional) Log a message to the console. Make sure you cast the buffer to a `string`.
9. Write a response on the connection back to the address returned by `ReadFrom`.


```go
func main() {
	conn, err := net.ListenPacket("udp", ":9001") 						// 1
	if err != nil { 													// 2
		log.Fatal(err)
	}
	defer conn.Close() 													// 3
	buf := make([]byte, 1024)											// 4

	for { 																// 5
		_, addr, err := conn.ReadFrom(buf) 								// 6
		if err != nil { 												// 7
			log.Fatal(err)
		}
		log.Printf("Received %s from %s", string(buf), addr) 			// 8
		conn.WriteTo([]byte("Hello from the UDP server"), addr) 		// 9
	}
}
```

## UDP client

A UDP client is almost identical to the TCP client, but you pass the `"udp"` protocol to the `Dial` function. A simple UDP client connects to the server with `Dial`, writes to the connection, and closes the connection:
1. Open the connection to the server with the `Dial` function. `Dial` returns a generic `net.Conn` interface for packet-oriented connections. It is a Reader and a Writer.
2. Handle any errors.
3. Close the connection.
4. Write to the connection with `Write`.

```go
func main() {
	conn, err := net.Dial("udp", ":9001") 			// 1
	if err != nil { 								// 2
		log.Fatal(err)
	}
	defer conn.Close() 								// 3
	conn.Write([]byte("Hello from UDP client")) 	// 4
}
```

## Network logging

The 12-factor app paradigm explains that you should treat logs as event streams. You can connect to a remote port that is designated for network logging and write logs there.

The following example is simple, but it demonstrates how to connect to a server, flush the network buffer when the connection closes, and create a network logger:
1. Establish a TCP connection on the listening port.
2. Close the connection. If a panic occurs, the network buffer is flushed to the destination so you don't lose important messages.
3. Define the log options.
4. Create the logger. The `New` function takes a `Writer`, prefix, and log flags.
5. Log a regular message.
6. Log a panic. Always use `Panicln` rather than `log.Fatal`, because the `Fatal` functions do not call the deferred functions---they immediately return with `os.Exit`. This means that the network buffer is never properly flushed. 

```go
func main() {
	conn, err := net.Dial("tcp", "localhost:1902")                  // 1
	if err != nil {
		panic(errors.New("Failed to connect to localhost:1902"))
	}
	defer conn.Close()                                              // 2

	flags := log.Ldate | log.Lshortfile                             // 3
	logger := log.New(conn, "[example]", flags)                     // 4
	logger.Println("This is a regular message")                     // 5
	logger.Panicln("This is a panic.")                              // 6
}
```