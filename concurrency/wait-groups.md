---
title: 'Using Wait Groups'
description: 'Learn how to manage goroutine synchronization effectively using wait groups in Go'
date: '2025-03-24'
category: 'Concurrency'
---

In Go, wait groups are a feature of the `sync` package that help manage concurrency and synchronize goroutines. They are particularly useful for waiting until a collection of goroutines have completed executing. This snippet shows how to use wait groups to manage goroutine synchronization effectively.

## Basic Wait Group Usage

Here's a basic example illustrating how to use a wait group to wait for multiple goroutines to finish:

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // Signal that this goroutine is done
	fmt.Printf("Processing task %d\n", id)
	// Simulate work.
	fmt.Printf("Task %d completed\n", id)
}

func main() {
	var wg sync.WaitGroup

	for i := 0; i < 3; i++ {
		wg.Add(1)        // Increment the wait group counter
		go worker(i, &wg) // Launch a goroutine
	}

	wg.Wait() // Block until all goroutines have finished
	fmt.Println("All tasks completed")
}
```

In this example, a simple worker function is executed concurrently. The `sync.WaitGroup` is used to ensure that the `main` function waits for all worker goroutines to complete before exit.

## Practical Usage with Error Handling

Sometimes tasks executed by goroutines can fail, and handling these errors is crucial. Here's how to capture errors and handle them gracefully:

```go
package main

import (
	"fmt"
	"sync"
)

const numWorkers = 5

func worker(id int, wg *sync.WaitGroup, errChan chan<- error) {
	defer wg.Done()

	// Simulate a task.
	if id%2 == 0 {
		errChan <- fmt.Errorf("task %d encountered an error", id)
		return
	}

	fmt.Printf("Task %d completed successfully\n", id)
}

func main() {
	var wg sync.WaitGroup
	errChan := make(chan error, numWorkers)

	for i := 0; i < numWorkers; i++ {
		wg.Add(1)
		go worker(i, &wg, errChan)
	}


	wg.Wait()
	close(errChan)

	for err := range errChan {
		if err != nil {
			fmt.Println("Error:", err)
		}
	}
	fmt.Println("All tasks completed")
}
```

## Best Practices

- Always call `wg.Done()` inside a goroutine to signal completion, ideally using `defer wg.Done()` at the start of the goroutine to ensure it gets called.
- Use `wg.Add()` before starting a goroutine to increment the counter, making sure all goroutines are accounted for.
- Always pass pointers to the wait group (`*sync.WaitGroup`) to goroutines to avoid copying.
- Properly close any channels used to communicate errors or data between goroutines.

## Common Pitfalls

- Forgetting to decrement the counter using `wg.Done()`, leading to deadlocks.
- Not using `wg.Wait()` on the main goroutine to block until all worker goroutines are complete.
- Accidentally over- or under-counting `wg.Add()`, which can lead to the `Wait()` call waiting indefinitely or returning too early.
- Failing to handle errors that occur within goroutines, which can lead to silent failures.

## Performance Tips

- Minimize the use of global variables accessible by multiple goroutines to avoid contention.
- Keep the critical sections (where goroutines access shared resources) as small as possible.
- Consider channel-based synchronization if you need to handle complex patterns of communication.
- Use buffered channels for error handling to prevent goroutines from blocking on sending errors.
