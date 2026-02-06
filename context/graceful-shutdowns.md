---
title: 'Graceful Shutdowns in Go'
description: 'Implementing graceful shutdowns in Go to ensure smooth termination of applications by cleaning up resources efficiently.'
date: '2025-03-24'
category: 'Context'
---

Graceful shutdowns are essential in applications to ensure resources are properly released, connections are closed, and ongoing operations are finished neatly before exiting. Go makes this easier with context cancellation and the `os/signal` package.

## Basic Graceful Shutdown Example

The following code demonstrates a simple server that can be gracefully shut down, handling incoming OS signals:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	server := &http.Server{Addr: ":8080", Handler: http.DefaultServeMux}

	// Listen for incoming OS signals.
	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)

	// Start server in a goroutine.
	go func() {
		fmt.Println("Starting data processing server...")
		if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatal(err)
		}
	}()

	// Block until we receive a signal.
	<-signalChan
	fmt.Println("Shutdown signal received")

	// Create a deadline for graceful shutdown.
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Shutdown the server gracefully.
	if err := server.Shutdown(ctx); err != nil {
		log.Fatal(err)
	}
	fmt.Println("Server gracefully stopped")
}
```

## Using signal.NotifyContext

Starting with Go 1.16, `signal.NotifyContext` simplifies graceful shutdown logic by creating a context that is canceled when specific signals are received.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	// Context canceled on SIGINT or SIGTERM.
	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	server := &http.Server{Addr: ":8080", Handler: http.DefaultServeMux}

	go func() {
		fmt.Println("Server listening on :8080")
		if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatal(err)
		}
	}()

	<-ctx.Done()
	fmt.Println("Shutdown signal received")

	shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	if err := server.Shutdown(shutdownCtx); err != nil {
		log.Fatal(err)
	}
	fmt.Println("Server gracefully stopped")
}
```

> See the official example [os/signal.NotifyContext](https://pkg.go.dev/os/signal#example-NotifyContext).

## Best Practices

- **Use Contexts**: Pass a context to handle timeouts, cancellations, and deadlines for ongoing operations during shutdown.
- **Ensure Cleanup**: Make sure all io.Writer/Reader and other resources are properly closed.
- **Handle All Signals**: Listen for all relevant signals that may indicate a termination request, like `SIGINT` and `SIGTERM`.
- **Timeouts**: Set an adequate timeout for graceful shutdown to avoid hanging for too long, but long enough to complete ongoing requests.
- **Proper Goroutine Management**: Ensure that spawned goroutines are aware of context cancellation to properly exit.

## Common Pitfalls

- **Ignoring Errors**: Don't ignore errors during shutdown; they may indicate resources weren't cleaned properly.
- **Hard Exits**: Avoid `os.Exit(1)` before cleaning up resources, as it will stop execution abruptly without cleanup.
- **Deadlock on Shutdown**: Ensure that signal handling and shutdown logic don't block each other, causing a deadlock.

## Performance Tips

- **Minimize Active Routines**: Close idle goroutines proactively to cut down shutdown time.
- **Optimized Timeouts**: Test and optimize the timeout based on empirical data from running applications.
- **Profiler Use**: Leverage Go's profiling tools to investigate any shutdown performance issues related to resource handling.