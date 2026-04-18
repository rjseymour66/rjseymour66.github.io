+++
title = 'UDP'
date = '2025-09-06T22:59:25-04:00'
weight = 20
draft = false
+++

UDP clients faster than TCP clients because they have the following advantages:
- Do not require `ACK` messages from the server to continue sending data.
- Do not establish connections
- Do not ensure packet order.
- Can easily broadcast and multicast, sending data packets to multiple recipients on a network.
- Let you implement your own reliability mechanism for your use case.

These reasons all reduce overhead and alleviates common network issues like backpressure.

## Caveats

UDP has the following disadvantages:
- Messages can get lost---UDP does not acknowledge each message and require retransmission.
- Messages can be received out of order.
- Sending many UDP messages can overwhelm a server if it cannot handle all the requests.

## Simple UDP server

To test UDP programs, you need a UDP server. Netcat is a simple command line utility that accepts simple test messages on the specified port and writes them to the console.

Run this command in a terminal to start a UDP (`-u`) server that listens (`-l`) continuously (`-k`) on port 1902:

```bash
nc -luk 1902
```

## Network logging

The 12-factor app paradigm explains that you should treat logs as event streams. You can connect to a remote port that is designated for network logging and write logs there.

The following example is simple, but it demonstrates how to connect to a server with a timeout, flush the network buffer when the connection closes, and create a network logger:
1. Define a timeout. In UDP, you add the timeout to account for how long it takes to send the message. UDP reads can block forever, so adding a timeout ensures the connection eventually closes.
2. Establish a UDP connection with a timeout on a listening port.
3. Close the connection. If a panic occurs, the network buffer is flushed to the destination so you don’t lose important messages.
4. Define the log options.
5. Create the logger. The `New` function takes a `Writer`, prefix, and log flags.
6. Log a regular message.
7. Log a panic. Always use `Panicln` rather than `log.Fatal`, because the `Fatal` functions do not call the deferred functions—they immediately return with `os.Exit`. This means that the network buffer is never properly flushed.

```go
func main() {
	timeout := 30 * time.Second                                         // 1
	conn, err := net.DialTimeout("udp", "localhost:1902", timeout)      // 2
	if err != nil {
		panic(errors.New("Failed to connect to localhost:1902"))
	}
	defer conn.Close()                                                  // 3

	flags := log.Ldate | log.Lshortfile                                 // 4
	logger := log.New(conn, "[example]", flags)                         // 5
	logger.Println("This is a regular message")                         // 6
	logger.Panicln("This is a panic.")                                  // 7
}
```

## Selective Retransmission

Also called Selective Acknowledgement (SACK), this is a technique you can implement in UDP where the receiver keeps track of which packets have been successfully received so that it can tell the sender exactly which packets its missing. This prevents the server from resending the same packets in case of packet loss.

[This repo](https://github.com/PacktPublishing/System-Programming-Essentials-with-Go/blob/main/ch10/selective-retransmissions/main.go) has an educational example that is not production-ready.