+++
title = 'Concurrency'
date = '2025-08-29T08:27:17-04:00'
weight = 50
draft = false
+++


Go's uses event-style _goroutines_ and _channels_ to provide an efficient concurrency model:

goroutine
: A function that runs independently of the function that started it. It shares the same thread, but has its own function stack.

channel
: A pipeline for sending and receiving data, much like a socket that runs inside your program and sends and accepts signals. Channels provide a way for goroutines to send data to their caller and one another.