---
title: 'Logging Levels and Configuration'
description: 'Understanding and configuring logging levels in Go for effective application logging'
date: '2025-03-24'
category: 'Logging'
---

Logging is a crucial part of application development and maintenance. Go provides several libraries for logging, including the standard `log` package and popular third-party libraries like `zap`. This guide covers how to use different logging levels and configure them to enhance your application's logging system.

## Basic Logging with `log`

The standard `log` package in Go provides basic logging functionality. Here's an example of setting up different logging levels and using them:

```go
package main

import (
	"log"
	"os"
)

var (
	InfoLogger  *log.Logger
	WarnLogger  *log.Logger
	ErrorLogger *log.Logger
)

func init() {
	file, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
	if err != nil {
		log.Fatal(err)
	}
	defer file.close()

	InfoLogger = log.New(file, "INFO: ", log.Ldate|log.Ltime|log.Lshortfile)
	WarnLogger = log.New(file, "WARN: ", log.Ldate|log.Ltime|log.Lshortfile)
	ErrorLogger = log.New(file, "ERROR: ", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
	InfoLogger.Println("This is informational log")
	WarnLogger.Println("This is a warning log")
	ErrorLogger.Println("This is an error log")
}
```

## Using `log/slog`

The `log/slog` package provides structured logging with built-in level support:

```go
package main

import (
	"log/slog"
	"os"
)

func main() {
	logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
	
	logger.Debug("Debug message")
	logger.Info("Info message")
	logger.Warn("Warning message")
	logger.Error("Error message")
}
```

## High-Performance Logging with `zap`

`zap` is a fast, structured logging library that provides both a low-level barebones logger and a higher-level sugared logger. Here is an example of configuring `zap`:

```go
package main

import (
	"go.uber.org/zap"
)

func main() {
	logger, _ := zap.NewProduction()
	defer logger.Sync()

	sugar := logger.Sugar()
	sugar.Infow("logged with zap",
		"level", "info",
		"example", "zap logging",
	)
	sugar.Warnw("This is a warning log",
		"example", "warning",
	)
	sugar.Errorw("An error occurred",
		"example", "error",
	)
}
```

## Best Practices

- **Choose the Right Library**: Use `zap` if performance is a priority, or the standard `log` package for simpler use cases.
- **Use Structured Logging**: Prefer structured logging (key-value pairs) for better organization and retrieval in log processing systems.
- **Configurable Logging Levels**: Allow dynamic configuration of logging levels to control verbosity without recompiling the application.

## Common Pitfalls

- **Ignoring Log Output Location**: Always specify the correct log output (file, stdout, etc.) to avoid missing log messages.
- **Overly Verbose Logging**: Avoid logging excessively at higher levels (info, debug) which can obscure critical errors.
- **Lack of Contextual Information**: Ensure logs provide enough context to assist debugging, typically by including relevant request or environment data.

## Performance Tips

- **Manage Log Rotation**: Implement log rotation to prevent enormous log files from degrading performance.
- **Use Efficient Formats**: Leverage efficient log formats like JSON offered by libraries like `zap` for high-volume logging.
- **Minimize Blocking**: In case of heavy logging, consider logging asynchronously or using batch logging to reduce application blocking.

