---
title: 'HTTP Error Handling'
description: 'Learn how to handle HTTP errors in Go using the net/http package'
date: '2025-03-24'
category: 'HTTP'
---

Handling HTTP errors efficiently is crucial for building robust web applications in Go. The `net/http` package provides a straightforward way to manage errors during HTTP requests and responses.

## Basic HTTP Error Handling

Here is an example demonstrating how to handle an HTTP request and respond with appropriate status codes:

```go
package main

import (
	"fmt"
	"log"
	"net/http"	
)

func handler(w http.ResponseWriter, r *http.Request) {
	// Simulating an error condition.
	if r.Method != http.MethodGet {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	// Process the request (for a valid method).
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Welcome to our server!")
}

func main() {
	http.HandleFunc("/", handler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Handling Client-Side HTTP Errors

Below is an example of handling errors when making HTTP requests from a client perspective:

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	resp, err := http.Get("http://example.com/nonexistent")
	if err != nil {
		fmt.Println("Error making request:", err)
		return
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		fmt.Println("Server returned non-200 status:", resp.Status)
		return
	}

	// Continue processing the response...
}
```

## Advanced HTTP Error Handling with Context

Using context for more advanced error handling and request management:

```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"time"
)

func requestWithTimeout(url string, timeout time.Duration) error {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()

	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return fmt.Errorf("creating request failed: %v", err)
	}

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return fmt.Errorf("request failed: %v", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return fmt.Errorf("server returned status: %s", resp.Status)
	}

	return nil
}

func main() {
	err := requestWithTimeout("http://example.com", 2*time.Second)
	if err != nil {
		fmt.Println("Request error:", err)
	} else {
		fmt.Println("Request successful")
	}
}
```

## Best Practices

- **Use Contexts**: Use context for request timeouts and cancellations to prevent hanging requests.
- **Proper Error Messages**: Clearly describe the error in server responses using `http.Error`.
- **Logging**: Log errors with sufficient detail to aid debugging without leaking sensitive information.

## Common Pitfalls

- **Ignoring Status Codes**: Always inspect HTTP status codes to handle non-2xx statuses properly.
- **Error Handling**: Avoid using `panic` for regular error handling; instead, return descriptive errors.
- **Misconfigured Timeouts**: Be careful with timeouts, as too short can cause false negatives and too long can lead to resource hogging.

## Performance Tips

- **Use Keep-Alive Connections**: Reuse HTTP connections to reduce latency.
- **Configure Timeouts**: Apply appropriate timeouts on client and server to improve responsiveness.
- **Error Responses**: Return minimal error pages to reduce bandwidth usage.

These techniques ensure that your Go applications can gracefully handle errors and remain robust under varying conditions.