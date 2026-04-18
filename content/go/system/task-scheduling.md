+++
title = 'Task Scheduling'
date = '2025-12-06T12:19:23-05:00'
weight = 40
draft = false
+++

Schedule tasks with concurrency and time package. Scheduling has this benefits:
- Run processes at off-peak hours to use resources efficiently.
- Create backups, updates, and maintenance.
- In distributed systems, manage when and how tasks are executed in parallel.
- Make sure tasks are performed at regular intervals.

## Create a scheduler

Define a `Job` type and a `Scheduler` type:
1. `Job` takes no arguments and returns no value.
2. `Scheduler` has a channel named `jobQueue` that stores and manages scheduled jobs.
3. The factory function creates a new scheduler. It creates a buffered channel of a given size so you can add jobs to `jobQueue` without blocking.

```go
type Job func()                             // 1

type Scheduler struct {                     // 2
	jobQueue chan Job
}

func NewScheduler(size int) *Scheduler {    // 3
	return &Scheduler{
		jobQueue: make(chan Job, size),
	}
}
```

Next, define the behavior of the scheduler. We want to run every job in its own goroutine:
1. The `Start` method ranges through all jobs in `jobQueue` and runs each in a goroutine.
2. `Schedule` sends a job in to `jobQueue`.
3. Because this happens in a goroutine, `Schedule` doesn't block execution for the duration of delay.

```go
func (s *Scheduler) Start() {                                       // 1
	for job := range s.jobQueue {
		go job()
	}
}

func (s *Scheduler) Schedule(job Job, delay time.Duration) {        // 2
	go func() {                                                     // 3
		time.Sleep(delay) 
		s.jobQueue <- job
	}()
}
```

Here is how you run the scheduler:
1. Create the new sheduler. This scheduler can run 10 concurrent jobs.
2. Define the `Schedule` method for the scheduler. This example uses an anonymous function that prints a message to the console after 5 seconds.
3. Start the scheduler.
4. `fmt.Scanln()` reads input from stdin. For this program, it reads Enter from the keyboard.

```go
func main() {
	scheduler := NewScheduler(10)                               // 1

	scheduler.Schedule(func() {                                 // 2
		fmt.Println("Job executed at", time.Now())
	}, 5*time.Second)

	go scheduler.Start()                                        // 3

	fmt.Println("Sheduler started. Press 'Enter' to exit.")     // 4
	fmt.Scanln()                                                // 5
}
```

## Timer signals

A `Ticker` lets you do something at regular intervals. You can combine this with a scheduler to execute a task at regular intervals. This example runs a task every second for 10 seconds:
1. Create a `Ticker`. A ticker has a send channel of type `Time` that accepts ticks from the ticker. This is how you define the interval between tasks.
2. Ensure the ticker turns off.
3. Create a `Timer`. This is the period of time that the scheduler runs. This timer runs for 10 seconds.
4. Ensure the timer turns off.
5. Use an infinte `for` loop and `select` statement to wait for events.
6. When `tick` receives a value from the ticker, it prints a message to the console.
7. When a value is returned on `timer`, it prints a message and returns, ending the program.

```go
func main() {
	ticker := time.NewTicker(1 * time.Second)       // 1
	defer ticker.Stop()                             // 2

	timer := time.NewTimer(10 * time.Second)        // 3
	defer timer.Stop()                              // 4

	for {                                           // 5
		select {
		case tick := <-ticker.C:                    // 6
			fmt.Println("Tick at", tick)
		case <-timer.C:                             // 7
			fmt.Println("Timer expired")
			return
		}
	}
}
```
