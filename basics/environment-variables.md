---
title: 'Environment Variables'
description: 'Learn how to effectively use environment variables in Go applications for configuration management'
date: '2025-03-24'
category: 'Tools'
---

Environment variables are a crucial aspect of managing configuration in Go applications, especially in cloud-based deployments. This guide demonstrates how to read and set environment variables in Go.

## Reading Environment Variables

Use `os.Getenv()` to retrieve the value of an environment variable. It returns an empty string if the variable does not exist.

```go
package main

import (
	"fmt"
	"os"
)

func main() {	
	host := os.Getenv("HOSTNAME")
	if host == "" {
		fmt.Println("HOSTNAME is not set")
		return
	}
	fmt.Println("Hostname:", host)
}
```

## Setting Environment Variables

You can set environment variables for the current process using `os.Setenv()`:

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	err := os.Setenv("RUNTIME_ENV", "production")
	if err != nil {
		log.Fatal(err)
	}

	appEnv := os.Getenv("RUNTIME_ENV")
	fmt.Println("Runtime Environment:", appEnv)
}
```

## Using `os.LookupEnv()` for Safe Retrieval

To safely determine if an environment variable is set and retrieve its value, use `os.LookupEnv()`:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	if dbPort, exists := os.LookupEnv("DB_PORT"); exists {
		fmt.Println("Database Port:", dbPort)
	} else {
		fmt.Println("DB_PORT is not set")
	}
}
```

## Best Practices

- Use `os.LookupEnv()` instead of `os.Getenv()` when you need to differentiate between an unset variable and an empty string.
- Use a consistent naming convention for environment variables and include an appropriate prefix, e.g., `APP_`, to avoid clashes.

## Common Pitfalls

- Forgetting to check if an environment variable is unset when using `os.Getenv()`.
- Hardcoding environment variable values in the source code, which may lead to issues when deploying to different environments.
- Using sensitive information in environment variables without securing access to them.

## Performance Tips

- Minimize the repeated access to environment variables by setting them once during the application startup and caching their values if used frequently.
- Avoid setting environment variables within performance-critical loops, as it incurs a syscall overhead.
- When running in containers, be mindful of the Docker and Kubernetes environment variable constraints, and consider optimizing their usage accordingly.