+++
title = 'Channels'
date = '2025-08-30T12:08:59-04:00'
weight = 20
draft = false
+++

Channels let you send data as messages from one goroutine to another. They are often described as sockets between goroutines in a single application, or pipes that share information asynchronously. It might be easiest to think of them as a mailbox where you send and receive data.

Channels have the following properties:
- Typed and can send structured data
- Bidirectional or unidirectional
- Short-lived or long-lived
- Can use multiple channels per app, with each channel working with a different data type

The idiomatic way to use channels is for passing data, distributing tasks, or communicating results.

## Channels basics

There are many ways to create and use channels.

### Creating a channel

Initialize a channel with `make`:

```go
var ch chan int
ch = make(chan int)
chTwo := make(chan []byte)
```

### Arrow operator

By default, channels are bidirectional, which means it can both send and receive data. To create a unidirectional channel---one that either sends or receives---use the arrow operator.

This table summarizes the arrow placement:

| Channel type | Example | Description             |
| :----------- | :------ | :---------------------- |
| send-only    | `ch <-` | Send data into `ch`.    |
| receive-only | `<-ch`  | Receive data from `ch`. |

{{< admonition "Caller's perspective" tip>}}
A channel's direction is determined by the caller's perspective. If the caller can only send data into a channel, then it is a "send-only" channel. If the caller can only receive data from a channel, then it is a "recieve-only" channel.
{{< /admonition >}}

### Send channels

A send channel is a channel that you send data into to retrieve by another go routine. This might seem backwards at first---a "send" channel sounds like it should send data from the channel. To make it more clear, think "the program sends data _into_ a send channel".

Send channels behave differently whether they are buffered or unbuffered. 

By default, channels are unbuffered. An unbuffered channel blocks until another goroutine is ready to receive the value. If you are going to send data into a channel, you need to have another channel ready to receive the data or the program ends in a deadlock.

When sending data into a channel, run it in a separate goroutine so it does not block execution to the receiving channel. For example, this code sends data to `ch` in the `main` goroutine. `main` cannot execute past `ch <- 100` because it blocks, resulting in a deadlock:

```go
func main() {
	ch := make(chan int)
	ch <- 100              // blocks until deadlock

	receiving := <-ch
}
```

To fix this, put `ch` in a goroutine so that execution can continue to the receiving goroutine:

```go
func main() {

	ch := make(chan int)
	go func() {
		ch <- 100
	}()

	val := <-ch
}
```



### Receive channels

A receiving channel is a channel that you receive data from. The data is sent into the channel by another goroutine. A receiving channel blocks until a value is sent, but that is usually the behavior that you want---the receiving channel is waiting for a signal that a task or other work is complete.

You can assign the value from a receiving channel, or you can discard it. To assign the value, place the receiving channel syntax (`<-ch`) on the right of an assignment expression. To discard it, just write the receiving channel syntax.

You want to assign the receiving channel when you need to perform additional work with its value. For example, this code assigns the value from an int channel and converts it to a string:
1. Create the sending channel.
2. Send data into the sending channel.
3. Receive the data from the sending channel and assign it to `done`
4. Convert the `int` to a `string`.

```go
func main() {
	ch := make(chan int)        // 1
	go func() {                 // 2
		ch <- 1
	}()

	done := <-ch                // 3
	str := strconv.Itoa(done)   // 4
	fmt.Printf("Type: %T, Val: %s\n", str, str)
}
```

{{< admonition "Remember the arrow" tip >}}
When you assign a value from a channel, remember to use the arrow on the receiving channel: `val := <-ch`. If you omit the arrow (`val := chan`), you are only assigning channel itself, which is a virtual memory address.
{{< /admonition >}}

#### Discard the value

To discard the value, end the program without assigning it to a variable. Use this pattern when the receiving channel does not need to preform any work with the channel value. For example, if the channel sends a signal that the program should exit, you do not need to store the value:

```go
func main() {

	ch := make(chan os.Signal)
	go func() {
		ch <- os.Kill
	}()
	<-ch
}
```

To see what `ch` receives, you can print it to the console:

```go
func main() {

	ch := make(chan os.Signal)
	go func() {
		ch <- os.Kill
	}()
	fmt.Println(<-ch)           // prints: "killed"
}
```

### Function arguments

When you pass a channel to a function, a best practice is to indicate whether the function sends or receives data on the channel. For example, this function sends data to the `out` channel:

```go
func readStdin(out chan<- []byte) {
	for {
		data := make([]byte, 1024)
		l, _ := os.Stdin.Read(data)
		if l > 0 {
			out <- data
		}
	}
}
```

## Unbuffered vs buffered

| Aspect                | Unbuffered                                                    | Buffered                                             |
| --------------------- | ------------------------------------------------------------- | ---------------------------------------------------- |
| Use case              | Hand-off communication, coordination, enforcing ordering      | Queues, pipelines, smoothing spikes, async behavior  |
| Send behavior         | Blocks until a receiver is ready                              | Blocks only when buffer is full                      |
| Receive behavior      | Blocks until a sender sends                                   | Blocks only when buffer is empty                     |
| Synchronization       | Provides implicit synchronization between sender and receiver | Decouples sender/receiver; not strictly synchronized |
| Capacity              | 0                                                             | N > 0                                                |
| When send unblocks    | Exactly when a receiver receives                              | When a receiver receives or buffer has space         |
| When receive unblocks | Exactly when a sender sends                                   | When buffer contains at least 1 value                |
| Backpressure behavior | Immediate backpressure                                        | Backpressure only when buffer is full                |
| Typical pattern       | `send <- value` waits for `x := <-send`                       | `send <- value` proceeds until buffer fills          |
| Example               | Worker must be ready to receive                               | Allow bursts of requests before workers catch up     |


## Unbuffered channels

By default, a channel in Go is unbuffered. This means that it holds only one value rather than a buffer of values. When you store a value in an unbuffered channel, it blocks until until that value is received from another goroutine. If you send two values to an unbuffered channel, then the second value blocks until the first is retrieved by another goroutine.

Unbuffered channels are useful in these scenarios:
- Guaranteed delivery of data.
- One-to-one communication between goroutines.
- Load balancing patterns that ensure work is evenly distributed.

Create an unbuffered channel without providing a capacity as the second argument to `make`:

```go
ch := make(chan int)
```

### Closing the channel

Go's garbage collector does not clean up channels---it only cleans up values that it is certain will not be used again. You need to manually clean up your unused channels to prevent memory leaks due to unneeded channels consuming system resources.

To close a channel, you need to use another channel that signifies when the work is complete. In Go, this channel is often named `done`.

A few things to remember when closing channels:
- Trying to send on a closed channel causes a panic.
- The sender should close the channel because only the sender knows when there is no more data to send. The sender is the goroutine that puts data in the channel.

#### Basic example

Here is a simple example of how to close a channel from the sender. The `sendStrings` function iterates over a slice of names, sends the values to a string channel, and then returns that channel as a receive-only channel. The goroutine closes the channel with `defer` so we call `close` right before the function returns.

`main` ranges over the channel to log the values to the console:

```go
func sendStrings() <-chan string {
	ch := make(chan string)
	names := []string{"Apple", "Banana", "Carrot", "Date"}

	go func() {
		defer close(ch)

		for _, v := range names {
			ch <- v
		}
	}()
	return ch
}

func main() {
	ch := sendStrings()

	for v := range ch {
		fmt.Println("got: ", v)
	}
}
```

#### Closing with done

A common Go idiom is using a `done <-chan struct{}` to notify goroutines that another channel has completed its work. We use the empty struct because it consumes zero memory.

This example implements the [basic example](#basic-example) and adds a notifier channel. The `sendStrings` function now accepts a receive-only channel so it can receive notification that indicates when the channel is complete:
1. This function creates the `out` channel, so it also closes it.
2. The `select` statement waits for either a value from the `name` slice or a value on the `done` channel. Because we do not need the value for the `done` channel, we discard it.

```go
func sendStrings(done <-chan struct{}) <-chan string {
	out := make(chan string)        // 1
	names := []string{"Apple", "Banana", "Carrot", "Date"}

	go func() {
		defer close(out)            // 1
		for _, v := range names {
			select {                // 2
			case out <- v:
				fmt.Printf("%v added to channel\n", v)
			case <-done:
				return
			}
		}
	}()
	return out
}
```

The `main` function reads from the `ch` returned from `sendStrings` until it reads "Carrot":
1. It creates the `done` channel, so it is responsible for closing it.
2. When the read value is "Carrot", it closes the channel. Because we passed the `done` channel into `sendStrings`, it triggers the `<-done` case in its `select` statement and returns before reading the values remaining in `ch`.

```go
func main() {
	done := make(chan struct{})     // 1
	ch := sendStrings(done)

	for v := range ch {
		fmt.Println("got: ", v)
		if v == "Carrot" {          // 2
			close(done)
		}
	}
}
```

{{< admonition "Closed channels" note >}}
Closing the channel in main triggers the `<-done` case in `sendStrings` because a closed channel always returns the zero type of the channel and a `false` value. Before the channel is closed, it sits empty. When the channel closes, `<-done` returns immediately, almost like a broadcast signal.
{{< /admonition >}}

#### Reading a closed channel

When you read from a closed channel, Go returns the zero type and `false` from the channel:

```go
func main() {
	upstreamStream := make(chan int)
	go func() {
		upstreamStream <- 10
	}()
	close(upstreamStream)
    
    // check if closed
    v, ok := <-upstreamStream
	if !ok {
		fmt.Println("channel is closed")
	} else {
		fmt.Println("got value:", v)
	}
}
```

## Buffered channels

A buffered channel is a channel that can hold more than one value---a buffer of values. Buffered channels are useful in the following scenarios:
- Asynchronous communiation between goroutines.
- Reducing contention when you have multiple producers so they don't have to wait for a receiver.
- Preventing deadlocks with buffering.
- Batch processing where data is produced and consumed at different rates.

To create a buffered channel, provide a capacity value as the second argument to `make`:

```go
ch := make(chan int, 2)
```

A buffered channel does not deadlock if another goroutine is not ready to accept its value---it blocks. So, if a buffered channel has a capacity of 1, then it can accept one value. If another goroutine wants to use the channel, it blocks until the channel is empty.

For example, the following code runs to completion because it uses a buffered channel:

```go
func main() {
	ch := make(chan int, 1)
	ch <- 1
	fmt.Println(<-ch)
	ch <- 2
	fmt.Println(<-ch)
}
```

If `ch` were an unbuffered channel, the program would deadlock at `ch <- 1` because there is not another goroutine ready to receive the value.

### Synchronization

Because a send on a buffered channel blocks until the channel is ready, buffered channesl are often used for synchronization. If your program already makes extensive use of channels, you can use buffered channels in place of mutex locks and unlocks. Any function using the channel has a "lock" on the channel. When the work is complete and the value is pulled from the channel, the channel is "unlocked" and another function can use the channel.

The following example demonstrates how you can lock and unlock a buffered channel with multiple goroutines:

```go
func worker(id int, lock chan bool) {
	log.Printf("%d wants the lock\n", id)
	lock <- true
	log.Printf("%d has the lock\n", id)
	<-lock
	log.Printf("%d is releasing the lock\n", id)
}

func main() {
	lock := make(chan bool, 1)
	for i := 1; i < 7; i++ {
		go worker(i, lock)
	}
	time.Sleep(3 * time.Second)
}
```

### Closing the channel

You do not have to explicitly close a buffered channel. Buffered channels are allocated on the heap. When `main` exits, the channel is unreachable. The Go garbage collector cleans up any unreachable heap objects.

However, if you need to signal to receiver that work is complete and the program will not send more values on the channel, you can close the buffered channel. This is common in the fan-out/fan-in pipeline patter.

## select

The `select` statement watches zero or more channels for an event. It behaves similarly to a `switch` statement for channels---it tests all cases simultaneously to see if any of them are ready to accept the task passed to the `select` statement:
- If one case can execute, `select` executes that case.
- If more than one case can execute, `select` randomly picks a case. It repeats this until there are no cases to execute.
- If no cases execute, `select` executes a `default` statement, if provided.

A `select` statement is a control flow mechanism, because it blocks if none of the statements can execute.

### Example

Here is a basic example that uses a `select` statement to echo console input back to the console. The `readStdin` function uses multiple channels to send and receive data:
1. The function accepts the `out` channel as an argument. `out` is a send-only channel---the program sends bytes into this channel.
2. There is an infinite `for` loop so we can read as long as needed.
3. The `data` channel is slice of bytes that can hold up to 1KB (1024 bytes).
4. `os.Stdin.Read` can read up to `len(data)` (1KB). It reads data from its file (`Stdin`) and stores it in the `data` channel. It returns the number of bytes read and an error. We discard the error.
5. When `l` is greater than `0` (any input was written to `Stdin`), the program sends the bytes stored in `data` to the `out` channel.

```go
func readStdin(out chan<- []byte) {     // 1
	for {                               // 2
		data := make([]byte, 1024)      // 3
		l, _ := os.Stdin.Read(data)     // 4
		if l > 0 {                      // 5
			out <- data
		}
	}
}
```

The main function uses a select statement to handle the channels:
1. The `echo` channel is bidirectional--it can send or receive a slice of bytes.
2. Pass `echo` to `readStdin`, and run it in a goroutine. This means that the program sends data from Stdin to `echo`. Now, the program needs another in `main` to receive the data.
3. Create a `select` statement in an infinite `for` loop. `select` will listen for an event on each of its `case` expressions until a case causes the loop to exit.
4. The first case creates a `buf` of type `[]byte` to receive data from `echo`. When `readStdin` writes data to `echo`, this case writes the contents of `echo` to `buf`. When this occurs, the `select` statement picks this case and writes the contents of `buf` to `Stdout`.
5. The second case calls `time.After` to send a timeout signal after 10 seconds. `time.After` waits for the specified time to elapse then returns a receive-only channel that contains the current time. We have no need for the current time, so we discard it. Even though we discard the value, the expression is treated as a truthy Boolean, which causes select to pick this case and return from the infinite loop.
   
   For additional details about `time.After`, see the [Go docs](https://pkg.go.dev/time#After).

```go
func main() {
	echo := make(chan []byte)                   // 1
	go readStdin(echo)                          // 2
	for {                                       // 3
		select {
		case buf := <-echo:                     // 4
			os.Stdout.Write(buf)
		case <-time.After(10 * time.Second):    // 5
			return
		}
	}
}
```

To make the `time.After` behavior more clear, the following `main` method is equivalent to the previous example:
1. Create a receive-only channel `sleepChannel`.
2. Use `sleepChannel` in the `select` case.
```go
func main() {

	sleepChannel := time.After(10 * time.Second)

	echo := make(chan []byte)
	go readStdin(echo)
	for {
		select {
		case buf := <-echo:
			os.Stdout.Write(buf)
		case <-sleepChannel:
			fmt.Println("Called sleepChannel")
			return
		}
	}
}
```

## time.Ticker (todo)

https://gobyexample.com/tickers

In Go, `time.NewTicker` is a function used to create a Ticker, which sends signals over a channel at regular, repeated intervals. It is the standard tool for executing periodic tasks like polling an API, logging system status, or running background maintenance.

### Key Characteristics

- **Periodic Execution**: Unlike `time.Timer`, which fires once, a Ticker repeats indefinitely until it is stopped.
- **The Channel (C)**: The ticker contains a receive-only channel, `ticker.C`, which transmits the current time at each interval.
- **Timing Accuracy**: It uses the monotonic clock and will adjust intervals or drop ticks if the receiver is too slow to keep up with the specified duration.
- **Resource Management**: You should call `ticker.Stop()` to release associated resources when the ticker is no longer needed.

{{< admonition "Stop()" note >}}
Since Go 1.23, unreferenced tickers can be garbage collected even if not stopped, but calling Stop() remains a best practice for compatibility and clarity. 
{{< /admonition >}}

```go
ticker := time.NewTicker(2 * time.Second)
defer ticker.Stop() // Ensure it's cleaned up

for t := range ticker.C {
    fmt.Println("Tick at", t)
}
```

## Ranging over channels

Range over channels just other collection types, but note that the channel `for range` does not return an index, only a value. The range keyword breaks the loop when the channel is closed:

```go
for val := range ch {
    fmt.Printf("%v", val)
}
```

When you range over an unbuffered channel, you need to close the channel or the program will deadlock. Here, a WaitGroup closes the channel:

1. Create a channel.
2. Create a WaitGroup.
3. Increment the WaitGroup counter as needed.
4. Run your functions in anonymous goroutines.
5. Defer a call to Done at the start of the goroutine.
6. Call your worker function.
7. In a separate goroutine, wait for the WaitGroups to finish, then close the channel.
8. Range over the channel.

```go
func main() {
	balls := make(chan string) 				// 1

	wg := sync.WaitGroup{} 					// 2
	wg.Add(2) 								// 3

	go func() { 							// 4
		defer wg.Done() 					// 5
		throwballs("red", balls) 			// 6
	}()

	go func() {
		defer wg.Done()
		throwballs("green", balls)
	}()

	go func() {
		wg.Wait() 							// 7
		close(balls) 						// 8
	}()

	for val := range balls {				// 9
		fmt.Printf("%s received!\n", val)
	}
}
```

## Signaling between channels


```go
func main() {
	signalChannel := make(chan bool)
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		defer wg.Done()
		fmt.Println("Goroutine 1 is waiting for a signal...")
		<-signalChannel
		fmt.Println("Goroutine 1 is now doing work...")
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		fmt.Println("Goroutine 2 is about to send a signal...")
		signalChannel <- true
		fmt.Println("Goroutine 2 sent a signal")
	}()
	wg.Wait()
	fmt.Println("Both goroutines have finished.")
}
```