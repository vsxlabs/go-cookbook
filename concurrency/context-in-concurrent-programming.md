---
title: 'Context in Concurrent Programming'
description: 'Understand how to use contexts in Go to manage and control concurrent operations effectively'
date: '2025-04-12'
category: 'Tools'
---

In Go, the `context` package is a powerful toolkit for managing the lifecycle of groutines, passing request-scoped values, and coordinating cancellation signals. This is especially useful in concurrent programming.

## Basic Context Usage

Here's how you can use a context to control the execution of goroutines:

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Processing stopped")
			return
		default:
			fmt.Println("Processing data...")
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go worker(ctx)

	time.Sleep(2 * time.Second)
	cancel() // Signals the context's cancellation
	time.Sleep(1 * time.Second) // Allow time for worker to stop
}
```

## Context with Timeout

Using context with a timeout can automatically cancel an operation after a certain interval:

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func slowOperation(ctx context.Context) {
	select {
	case <-time.After(5 * time.Second): // Simulate a long task.
		fmt.Println("Data processing completed")
	case <-ctx.Done(): // Cancels if timeout occurs.
		fmt.Println("Processing timed out:", ctx.Err())
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	slowOperation(ctx)
}
```

## Context with Value

You can pass request-scoped values using contexts. Use a custom type for keys to avoid collisions:

```go
package main

import (
	"context"
	"fmt"
)

// Define an unexported type for context keys to avoid collisions.
type contextKey string

const taskIDKey contextKey = "taskID"

func worker(ctx context.Context) {
	if taskID, ok := ctx.Value(taskIDKey).(int); ok {
		fmt.Printf("Processing task: %d\n", taskID)
	} else {
		fmt.Println("No task ID found in context")
	}
}

func main() {
	ctx := context.WithValue(context.Background(), taskIDKey, 42)
	worker(ctx)
}
```

## Best Practices

- **Use contexts**: Make sure to pass a `context.Context` to long-running functions. Avoid not using it with goroutines that may block or take a long time.
- **Propagate context carefully**: Always use the context directly from the function signature instead of creating new contexts unless necessary.
- **Cancel contexts**: Ensure you call the `cancel` function returned by context creation functions to avoid context leaks.

## Common Pitfalls

- **Ignoring context cancellation**: Failing to check `<-ctx.Done()` in goroutines running under a context can lead to resource leaks.
- **Misusing context values**: Context should be used to carry deadline, cancellation signal, or request-scoped data, not for passing optional parameters.
- **Nested context timeouts**: Incorrectly nesting timeouts can lead to unexpected cancellations; always create with adequate understanding.

## Performance Tips

- **Minimize context creation**: Context creation and cancellation can become a source of overhead; reuse context as much as possible.
- **Avoid passing unnecessary data**: Minimize data packed in the context to what's absolutely necessary for the operation.
- **Profile blocking operations**: Use context to manage operations and profile their blocking behavior to ensure responsiveness.