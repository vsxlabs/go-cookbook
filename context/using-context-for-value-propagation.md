---
title: 'Using Context for Value Propagation'
description: 'Learn how to use Go contexts to propagate values within your application'
date: '2025-03-24'
category: 'Context'
---

The Go `context` package is a powerful tool used to control cancellations and timeouts, but it can also be utilized to propagate request-scoped values across API boundaries and between processes. This snippet demonstrates how to propagate values using contexts.

## Basic Context Value Propagation

Here's how you can pass values using Go's context:

```go
package main

import (
	"context"
	"fmt"
)

// Define a custom key type to avoid collisions.
type contextKey string

const sessionIDKey contextKey = "sessionID"

func main() {
	ctx := context.Background()
	ctx = context.WithValue(ctx, sessionIDKey, 42)

	processRequest(ctx)
}

func processRequest(ctx context.Context) {
	sessionID, ok := ctx.Value(sessionIDKey).(string)
	if !ok {
		fmt.Println("sessionID not found in context")
		return
	}
	fmt.Printf("Processing request for session ID: %d\n", sessionID)
}
```

## Custom Key Types for Context

Instead of using primitive types or strings for context keys, it's better practice to define a custom key type to avoid potential collisions:

```go
package main

import (
	"context"
	"fmt"
)

type contextKey string

const sessionKey contextKey = "sessionID"

func main() {
	ctx := context.Background()
	ctx = context.WithValue(ctx, sessionKey, 42)

	processRequest(ctx)
}

func processRequest(ctx context.Context) {
	sessionID, ok := ctx.Value(sessionKey).(int)
	if !ok {
		fmt.Println("sessionID not found in context")
		return
	}
	fmt.Printf("Processing request for session ID: %d\n", sessionID)
}
```

## Nested Context Values

You can nest context values to pass multiple pieces of information:

```go
package main

import (
	"context"
	"fmt"
)

type contextKey string

const (
	userIDKey  contextKey = "userID"
	userRoleKey contextKey = "userRole"
)

func main() {
	ctx := context.Background()
	ctx = context.WithValue(ctx, userIDKey, 42)
	ctx = context.WithValue(ctx, userRoleKey, "admin")

	processRequest(ctx)
}

func processRequest(ctx context.Context) {
	userID, ok := ctx.Value(userIDKey).(int)
	if !ok {
		fmt.Println("userID not found in context")
		return
	}
	userRole, _ := ctx.Value(userRoleKey).(string)
	fmt.Printf("Processing request for user ID: %d with role: %s\n", userID, userRole)
}
```

## Best Practices

- **Immutable Design:** Treat contexts as immutable; always create a new context using `context.WithValue` rather than modifying an existing one.
- **Conciseness:** Store only necessary values within the context to keep it lightweight.
- **Avoid Abuse:** Do not use context for passing optional parameters; reserve it for highly relevant information that travels across API/infrastructure boundaries.

## Common Pitfalls

- **Key Collisions:** Using string or primitive types as keys which can lead to unintended key collisions.
- **Unintended Mutability:** Assuming you can mutate context values; they're designed to be immutable.
- **Overusing Context:** Overloading context with too many values, which can make the application difficult to maintain.

## Performance Tips

- **Use Custom Types for Keys:** Improves type safety and performance by avoiding key collisions and reducing type assertion costs.
- **Minimal Data in Context:** Keep values passed through context minimal to reduce memory overhead.
- **Avoid Large Values:** Large context values can significantly increase memory usage and slow down context propagation.