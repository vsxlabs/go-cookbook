---
title: 'Logging with Context'
description: 'Learn how to implement structured logging with context in Go applications for better traceability and debugging'
date: '2025-03-24'
category: 'Logging'
---

Structured logging with context in Go can significantly improve traceability and make debugging easier by providing additional metadata with each log entry. Starting with Go 1.21, the standard library includes the powerful `slog` package for structured logging.

## Using Slog with Context

The `slog` package is part of the Go standard library and provides modern structured logging capabilities with excellent performance.

### Setting Up Slog

Since `slog` is part of the standard library, no additional installation is needed. Here's a simple example:

```go
package main

import (
	"context"
	"log/slog"
	"os"
)

func main() {
	// Create a JSON logger.
	logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelInfo,
	}))

	// Set as default logger.
	slog.SetDefault(logger)

	// Add default attributes to all log entries.
	baseLogger := logger.With(
		"app", "example",
		"env", "production",
	)

	// Define a custom key type to avoid collisions.
	type contextKey string
	const correlationIDKey contextKey = "correlation_id"

	// Create a context with a correlation ID.
	ctx := context.WithValue(context.Background(), correlationIDKey, "12345")
	logWithContext(ctx, baseLogger)
}

func logWithContext(ctx context.Context, logger *slog.Logger) {
	type contextKey string
	const correlationIDKey contextKey = "correlation_id"
	correlationID := ctx.Value(correlationIDKey)

	// Create a logger with correlation ID
	l := logger.With("correlation_id", correlationID)

	l.Info("Starting processing")
	// Simulate processing
	l.LogAttrs(ctx, slog.LevelInfo, "Loading data",
		slog.String("step", "load data"))
	l.LogAttrs(ctx, slog.LevelInfo, "Processing data",
		slog.String("step", "process data"))
	l.Info("Processing completed")
}
```

### Using Context in HTTP Handlers

Here's how to integrate `slog` into HTTP handlers for request logging:

```go
package main

import (
	"context"
	"log/slog"
	"net/http"
	"os"
)

func main() {
	// Initialize JSON logger.
	logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelInfo,
	}))
	slog.SetDefault(logger)

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Define a custom key type to avoid collisions.
		type contextKey string
		const requestIDKey contextKey = "request_id"

		// Extract or generate a request ID for tracing
		requestID := r.Header.Get("X-Request-ID")
		if requestID == "" {
			requestID = "default-id"
		}

		// Attach the request ID to context.
		ctx := context.WithValue(r.Context(), requestIDKey, requestID)

		// Create request-scoped logger
		reqLogger := logger.With(
			"request_id", requestID,
			"path", r.URL.Path,
			"method", r.Method,
		)

		handleRequest(ctx, reqLogger, w, r)
	})

	if err := http.ListenAndServe(":8080", nil); err != nil {
		slog.Error(err)
		os.Exit(1)
	}
}

func handleRequest(ctx context.Context, logger *slog.Logger, w http.ResponseWriter, r *http.Request) {
	logger.InfoContext(ctx, "Handling request")
	// Simulate processing
	w.Write([]byte("Request handled"))
	logger.InfoContext(ctx, "Request processed")
}
```

## Best Practices

- Use structural logging to include metadata, which aids in debugging and tracing.
- Attach unique identifiers (e.g., request ID, user ID) to logs using context.
- Keep log output consistent and meaningful by following a logging format across your application.
- Centralize logging setup to easily adjust configurations and logging levels as needed.

## Common Pitfalls

- Failing to propagate the context through the application functions and layers might lose critical tracing information.
- Overloading context with excessive data can lead to performance issues.
- Logging sensitive information without appropriate care can lead to security risks.

## Performance Tips

- Use asynchronous logging if supported by the library to reduce performance overhead by avoiding I/O blocking operations.
- Consider the verbosity level of the logs, adjusting logging level during development and production to avoid unnecessary performance overhead.
- Regularly rotate and archive logs to prevent log files from becoming too large, which can negatively impact performance.