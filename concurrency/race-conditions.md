---
title: 'Race Conditions in Go'
description: 'Learn how to identify and handle race conditions in Go using standard library tools.'
date: '2025-03-24'
category: 'Tools'
---

Race conditions occur in concurrent programming when multiple goroutines access shared data simultaneously and at least one of them modifies the data. This can lead to unpredictable program behavior. Go provides several tools and constructs to handle race conditions effectively.

## Identifying Race Conditions

The Go runtime includes a race detector to help identify race conditions in your code. You can enable it when building or testing:

```shell
go run -race main.go
```

## Example of a Race Condition

Consider this simple function that increments a counter in multiple goroutines:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var value int
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			value++
		}()
	}

	wg.Wait()
	fmt.Println("Value:", value)
}
```

When run, this program may not print `Data value: 10` consistently due to a race condition on the `data` variable.

## Fixing Race Conditions

### Using Mutex

A `sync.Mutex` can be used to control access to shared data:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var value int
	var wg sync.WaitGroup
	var mu sync.Mutex

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			mu.Lock()
			value++
			mu.Unlock()
		}()
	}

	wg.Wait()
	fmt.Println("Value:", value)
}
```

### Using Channels

Channels can also be used to avoid race conditions by centralizing data access:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	const numGoroutines = 10

	dataCh := make(chan int, numGoroutines)
	var wg sync.WaitGroup

	for i := 0; i < numGoroutines; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			dataCh <- 1
		}()
	}

	go func() {
		wg.Wait()
		close(dataCh)
	}()

	total := 0
	for v := range dataCh {
		total += v
	}
	fmt.Println("Data value:", total)
}
```

## Best Practices

- Use `sync.Mutex` or `sync.RWMutex` to protect critical sections that modify shared state.
- Use channels when sharing immutable data and for coordinating goroutines.
- Limit the scope of locks to minimize contention and deadlocks.

## Common Pitfalls

- Forgetting to unlock a `Mutex` with `defer`, leading to deadlock.
- Using a `Mutex` when a `RWMutex` could provide better read performance.
- Creating race conditions by assuming goroutines execute in a predictable order.

## Performance Tips

- Prefer `sync.RWMutex` when reads outnumber writes to reduce contention.
- Use buffered channels to reduce synchronization overhead when using channel-based approach.
- Minimize the duration of locked sections to enhance concurrency.
- Combine atomic operations with locks for frequently accessed counters (e.g., `sync/atomic`).