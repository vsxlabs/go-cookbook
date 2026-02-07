---
title: 'Understanding Go Scheduler'
description: 'Explore how the Go scheduler works and how it manages goroutines efficiently.'
date: '2025-03-24'
category: 'Runtime'
---

Go's scheduler is a key component that enables efficient execution and management of goroutines. Unlike traditional OS-level threads, goroutines are managed by the Go runtime, which allows for more lightweight concurrency.

## Basics of Go Scheduler

Go's runtime uses an M:N scheduling model where M goroutines are multiplexed onto N OS threads, with GOMAXPROCS controlling the maximum number of OS threads that can execute Go code simultaneously.

### Example: Setting GOMAXPROCS

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func printGreetings(wg *sync.WaitGroup, id int) {
	defer wg.Done()
	fmt.Printf("Hello from goroutine %d\n", id)
}

func main() {
	runtime.GOMAXPROCS(2) // Set the maximum number of OS threads to 2
	var wg sync.WaitGroup

	for i := 1; i <= 5; i++ {
		wg.Add(1)
		go printGreetings(&wg, i)
	}
	
	wg.Wait()
}
```

## Cooperative Multitasking

Go uses cooperative multitasking for goroutines, where goroutines yield control to the scheduler during specific operations, such as waiting on I/O or using `runtime.Gosched()`.

### Example: Cooperative Scheduling with `runtime.Gosched()`

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func task(wg *sync.WaitGroup, id int) {
	defer wg.Done()
	for i := 0; i < 3; i++ {
		fmt.Printf("Goroutine %d executed\n", id)
		runtime.Gosched() // Yield to other goroutines
	}
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go task(&wg, i)
	}
	
	wg.Wait()
}
```

## Work Stealing

The Go scheduler uses a work-stealing algorithm to balance load across multiple processors. If a processor has no work to do, it can steal work from another processor.

## Best Practices

- Set `GOMAXPROCS` according to your application's concurrency requirements and the number of logical CPU cores available (`runtime.NumCPU()`).
- Use synchronization primitives like channels and `sync.WaitGroup` to manage goroutine lifecycles efficiently.
- Avoid relying on goroutine scheduling order for program correctness—it can vary between runs.

## Common Pitfalls

- Overloading the scheduler with excessive goroutines (e.g. thousands, tens of thousands) without synchronization, causing resource contention.
- Assuming that goroutines will always run in the order they are created.
- Forgetting to decrease wait groups or failing to synchronize goroutines leading to deadlocks.

## Performance Tips

- Keep goroutine creation to a necessary minimum, especially in tight loops, to avoid overwhelming the scheduler.
- Profile your application using tools like `go tool pprof` to understand and optimize goroutine performance.
- Use `runtime.Gosched()` judiciously to yield control only when necessary—it can introduce unnecessary context switching if overused.