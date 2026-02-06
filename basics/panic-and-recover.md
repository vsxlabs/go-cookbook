---
title: 'Panic and Recover'
description: 'Understand how to use panic and recover to handle unexpected errors in Go applications'
date: '2025-03-24'
category: 'Tools'
---

In Go, `panic` and `recover` mechanisms are often used for handling unexpected errors and ensuring graceful recovery in applications. While they should not replace conventional error handling, they can be useful in certain situations.

## Basic Panic and Recover Usage

Below is an example that illustrates the use of `panic` and `recover` in a Go program:

```go
package main

import (
	"fmt"
)

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
		}
	}()

	fmt.Println("Starting the program")
	causePanic()
	fmt.Println("This line won't be reached because of the panic")
}

func causePanic() {
	panic("Unable to proceed")
}
```

## Using Panic and Recover with HTTP Handlers

Here's how you might use `panic` and `recover` to ensure your HTTP server does not crash:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", safeHandler(exampleHandler))
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}

func exampleHandler(w http.ResponseWriter, r *http.Request) {
	panic("Test panic")
}

func safeHandler(h http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		defer func() {
			if err := recover(); err != nil {
				fmt.Fprintf(w, "Recovered from panic: %v", err)
				// Code continues to be executed.
			}
		}()
		h(w, r)
	}
}
```

## Best Practices

- **Minimal Usage**: Use `panic` and `recover` sparingly. They are intended for serious errors that are not meant to be recovered during normal operation.
- **Log Panics**: When using `recover`, always log the error. This can help identify the cause of the panic during debugging.
- **Graceful Shutdown**: Ensure resources like files, network connections, and other systems are properly closed with `defer` even when a panic occurs.
- **Separate Error Handling**: Use standard error handling practices for expected errors. Reserved `panic` for truly exceptional cases.

## Common Pitfalls

- **Overuse of Panic**: Relying on panic for normal error handling instead of returning error values or using standard error handling patterns.
- **Ignoring Recovery**: Ensure that the recovery block responsibly handles the error and makes necessary cleanup or logging.  
- **Non-Recovery Scenarios**: Using `recover` outside of a `defer` function will not catch a panic.

## Performance Tips

- **Avoid Unnecessary Panics**: Since recovering from a panic is relatively expensive, avoid unnecessary panics to maintain performance.
- **Limit Scope**: Keep panic and recover confined to narrow scopes rather than across large sections of code to minimize the impact.
- **Properly Profile**: Especially in high-performance systems, benchmark and profile the use of panic and recover to understand their impact on performance. 

By effectively integrating and managing `panic` and `recover`, Go developers can improve the resilience of their applications for exceptional conditions.