---
title: 'Worker Pools in Go'
description: 'Understand and implement worker pools in Go for efficient concurrent processing.'
date: '2025-03-24'
category: 'Tools'
---

Worker pools in Go are a powerful concurrency pattern that enables efficient parallel execution of tasks. It's particularly useful for managing a fixed number of goroutines to process a large batch of jobs without overwhelming system resources.

## Basic Worker Pool Implementation

Here is a basic example of a worker pool using Go channels and goroutines.

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for j := range jobs {
		fmt.Printf("Worker %d processing task %d\n", id, j)
		// Simulate work by multiplying the number by 2.
		results <- j * 2
		fmt.Printf("Worker %d completed task %d\n", id, j)
	}
}

func main() {
	const numWorkers = 3
	const numJobs = 5

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)
	var wg sync.WaitGroup

	// Start workers.
	for w := 0; w < numWorkers; w++ {
		wg.Add(1)
		go worker(w, jobs, results, &wg)
	}

	// Send jobs to the workers.
	for j := 0; j < numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Wait for all workers to finish.
	wg.Wait()
	close(results)

	// Collect results.
	for result := range results {
		fmt.Println("Result:", result)
	}
}
```

### Go 1.25+ Usage with WaitGroup.Go

Starting with Go 1.25, you can use `WaitGroup.Go` to simplify worker pool management:

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Printf("Worker %d processing task %d\n", id, j)
		// Simulate work by multiplying the number by 2.
		results <- j * 2
		fmt.Printf("Worker %d completed task %d\n", id, j)
	}
}

func main() {
	const numWorkers = 3
	const numJobs = 5

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)
	var wg sync.WaitGroup

	// Start workers.
       for w := 0; w < numWorkers; w++ {
	       wg.Go(func() { worker(w, jobs, results) })
       }

	// Send jobs to the workers.
	for j := 0; j < numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	wg.Wait()
	close(results)

	// Collect results.
	for result := range results {
		fmt.Println("Result:", result)
	}
}
```

In this version, the `WaitGroup` counter is managed automatically, and the worker function does not need to receive a `*sync.WaitGroup` parameter.

## Best Practices

- Use buffered channels to avoid blocking sends when the number of jobs is small compared to the number of workers.
- Use a `sync.WaitGroup` to ensure all workers complete their tasks before closing the results channel.
- Ensure proper closing of the jobs channel once all jobs have been dispatched.
- Consider error handling mechanisms for failed tasks within workers.
- Dynamically adjust the number of workers based on available system resources and/or limits (for example, when doing logs of parallel requests).

## Common Pitfalls

- Forgetting to close the jobs channel, leading to deadlock.
- Not using a `sync.WaitGroup`, making it harder to coordinate the completion of all tasks.
- Starting too many goroutines, which might lead to resource exhaustion.
- Blocking on results channel reading due to an incorrect number of expected results.
- Not handling errors inside worker function properly, which can make debugging difficult.

## Performance Tips

- Use a pool of workers sized according to the number of available CPU cores to maximize efficiency.
- Profile your worker pool to identify bottlenecks, such as I/O or CPU-bound operations.
- Use more sophisticated techniques like job prioritization or load balancing across workers for advanced requirements.
- Avoid starting more goroutines than necessary by considering job distribution and worker feedback.
- Minimize global state access to reduce contention between workers.
