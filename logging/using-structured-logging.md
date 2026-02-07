---
title: 'Using Structured Logging'
description: 'Learn how to implement structured logging in Go using the built-in slog package.'
date: '2025-03-24'
category: 'Logging'
---

Structured logging is a method of logging where log entries are formatted in a way that machines can easily parse them. Using structured logging enables better log analysis and easier querying of logs. Go provides built-in support for structured logging through the `log/slog` package, introduced in Go 1.21.

## Basic Structured Logging with slog

Below is an example demonstrating how to use the `slog` package for structured logging.

```go
package main

import (
	"log/slog"
	"os"
)

func main() {
	// Initialize default logger
	logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
	slog.SetDefault(logger)

	// Structured logging
	slog.Info("User logged in",
		"event", "user_login",
		"user", "johndoe",
	)

	// Another structured log entry
	slog.Warn("File upload completed with warnings",
		"event", "file_upload",
		"user", "johndoe",
		"size", 3456,
	)
}
```

## Output Example

When you run the previous code with `NewTextHandler`, the structured logs will be in key=value text format:

```
time=2009-11-10T23:00:00.000Z level=INFO msg="User logged in" event=user_login user=johndoe
time=2009-11-10T23:00:00.000Z level=WARN msg="File upload completed with warnings" event=file_upload user=johndoe size=3456
```

If you want JSON output, use `NewJSONHandler` instead of `NewTextHandler`.

## Best Practices

- Use a structured logging library, like `logrus`, to avoid manually formatting JSON.
- Consistently use fields to capture important context about log events (e.g., user IDs, operation types).
- Set log levels appropriately to avoid unnecessary verbosity in production environments. 

## Common Pitfalls

- Forgetting to set the output of logs, resulting in logs being directed to stderr where they might be missed or ignored.
- Using plain log messages without any structured fields can make searching and filtering difficult.
- Ignoring performance considerations when logging extensively in high-throughput applications.

## Performance Tips

- Buffer log output to minimize impact on application performance.
- Avoid excessive logging in hot paths to prevent bottlenecks.
- Consider asynchronous logging to reduce latency.
