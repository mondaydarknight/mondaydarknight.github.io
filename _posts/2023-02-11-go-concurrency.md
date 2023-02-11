---
layout: post
title: Go concurrency overview
description: Concurrency synchronizations in Go
location: Taiwan
tags:
- Go
- Concurrency
---

## Go concurrency overview
Go is a concurrent language that provide a set of mechanisms to allow executing several computations concurrently and communicate between each concurrent procedures.

There's a misconception about concurrency and parallelism, explain these concepts as follows.

### Concurrency vs parallelism
`Concurrency` means that multiple tasks can be executed in an overlapping time period in no specific order. It can be broadly understood as multi-threading that allows numerous computations can be performed within a single processor.

> Concurrency is about dealing with lots of things at once.
Read more about [Concurrency is not parallelism](https://go.dev/blog/waza-talk)

`Parallelism` means that multiple tasks can be executed at the same time with multiple computing resources simultaneously (e.g. multi-core processor).

> Parallelism is about doing lots of things at once.

Concurrency refers to how a single CPU can make progress on multiple tasks at the same time.

Paralleism refers to how an application can parallelize the execution of a single task by splitting the task into subtasks which can be completed in parallel.

Read more about [concurrency/parallelism combinations](https://jenkov.com/tutorials/java-concurrency/concurrency-vs-parallelism.html).

### Goroutine
Go is a lightweight thread (also called [green trhead](https://en.wikipedia.org/wiki/Green_thread)) maintained and scheduled by language runtime. It's worth mentioning that the cost of memory consumption and context switching of goroutine is much lesser than OS threads, so it's not a problem to run tens of thousands goroutines at the same time as long as CPU and memory available.
Go's runtime uses resiable stacks to minimize the overhead of CPU and memory consumption to make these instructions cheap per function call.

Just use the keyword `go` followed by a function call that will be executed in a newly created goroutine.
```go
func Say(n int, s string) {
	for i := 0; i < n; i++ {
		fmt.Println(s)
		time.Sleep(time.Second)
	}
}

func main() {
	go Say(5, "hello")
	go Say(5, "world")
	time.Sleep(10 * time.Second)
}
```
Note the reason that uses `time.Sleep` is to make the main program wait for the goroutine because executions of goroutines is independant of the main program, so goroutine starts before the program exits since the goroutine doesn't have time to execute.

#### Concurrency synchronization
Since concurrent computations may share resources, so there may be [race condition](https://en.wikipedia.org/wiki/Race_condition) when two or more goroutines access the same memory location concurrently, leading to undefined or unpredictable results.
The ways to solve the problem are called concurrency synchronization by determining when to start/end, block/unblock and distribute the workload among concurrent computations.

There are several ways to implement concurrency synchronization techniques such as traditional mutex locks, channels (mentioned below) to protect the shared memory to protect data races.

For example, here we use the [`sync.WaitGroup`](https://pkg.go.dev/sync#WaitGroup) to synchronize the executions between two new goroutines and main goroutine. `sync.WaitGroup` are used to run multiple goroutines and block the calling thread until they're all complete, it's recommended to use a fan-out pattern to run a bunch of goroutines to process data and wait until all of them complete.

```go
var wg sync.WaitGroup

func Say(n int, s string) {
	for i := 0; i < n; i++ {
		fmt.Println(s)
		time.Sleep(time.Second)
	}
	wg.Done()
}

func main() {
	wg.Add(2)
	go Say(5, "hello")
	go Say(5, "world")
	wg.Wait()
}
```

### Channel
Channel is a communication mechanism that allows multiple goroutines to exchange data. It acts as a concurrency synchronization technique as an internal FIFO (first-in, first-out) queue within a program.
- `chan T` represents a bidirectional channel type, allow both receiving values from and sending values.
- `chan<-T` represents a send-only channel type only allow sending values to the chanenl.
- `<-chan T` represents a receive-only channel type only allow receiving values from the chanenl.

#### Types of channels
There're two types of channels: unbuffered channels and buffered channels.

`Unbuffered channel`
Unbuffered channel initially has no capacity requires both goroutines to be ready to make exchange. When a goroutine attempts to send a message to an unbuffered chanenl, if there's no goroutine waiting to receive, the channel will block the sending goroutine until the receiving goroutine has received, vice versa.

```go
func main() {
	n := 10
	c := make(chan int)
	defer close(c)
	go func(c <-chan int) {
		for i := range c {
			fmt.Println(i)
			time.Sleep(time.Second)
		}
	}(c)
	for i := 1; i <= n; i++ {
		c <- i
		fmt.Println("filled")
	}
	time.Sleep(3 * time.Second)
}
```

`Buffered channel`:
Buffered chanenl initially has capacity that allowing specify a fixed length of buffer capacity to send that number of data at once.
When a goroutine attempts to send a message to a buffered chanenl, if the chanenl is full, the channel will block the goroutine until the buffer becomes available.
```go
func main() {
	n := 10
	c := make(chan int, 3)
	defer close(c)
	go func(c <-chan int) {
		for i := range c {
			fmt.Println(i)
			time.Sleep(time.Second)
		}
	}(c)
	for i := 1; i <= n; i++ {
		c <- i
		fmt.Println("filled")
	}
	time.Sleep(3 * time.Second)
}
```
