---
title: 'Input Validation'
description: 'Learn how to perform input validation in Go to ensure data integrity and security.'
date: '2025-03-24'
category: 'Tools'
---

Input validation is crucial in ensuring that data entering your application is both correct and safe. In Go, there are several methods and packages that can help validate input effectively.

## Basic Input Validation Using `regexp`

One of the simple ways to validate input in Go is by using regular expressions with the `regexp` package. This is suitable for pattern-based validation.

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	email := "example@example.com"
	emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
	if emailRegex.MatchString(email) {
		fmt.Println("Email address is valid.")
	} else {
		fmt.Println("Email address is invalid.")
	}
}
```

## Using `validator` Package

The third-party package `github.com/go-playground/validator/v10` is a widely-used library for input validation, offering a variety of built-in validation tags.

```go
package main

import (
	"fmt"
	"github.com/go-playground/validator/v10"
	"log"
)

type User struct {
	Name  string `validate:"required"`
	Age   int    `validate:"gte=0,lte=130"`
	Email string `validate:"required,email"`
}

func main() {
	v := validator.New()

	user := User{
		Name:  "Alice",
		Age:   25,
		Email: "alice@example.com",
	}

	err := v.Struct(user)
	if err != nil {
		if _, ok := err.(*validator.InvalidValidationError); ok {
			log.Fatal(err)
		}
		for _, err := range err.(validator.ValidationErrors) {
			fmt.Println("Validation Error:", err)
			log.Fatal(err)
		}
	} else {
		fmt.Println("User input is valid!")
	}
}
```

## Best Practices

- **Layered Validation**: Conduct input validation at multiple layers of your application (e.g., HTTP handlers, service layers).
- **Use Third-party Libraries**: Leverage libraries like `validator` for complex validation needs.
- **Regular Expressions**: Use regular expressions judiciously, as they can be complex and lead to performance issues if not optimized.
- **Input Sanitization**: Sanitize input to prevent injection attacks.

## Common Pitfalls

- **Over-reliance on Client-side Validation**: Always validate server-side in addition to client-side validation logic.
- **Ignoring Edge Cases**: Consider edge cases like maximum length, unexpected symbols, or binary characters.
- **Not Handling Errors Appropriately**: Ensure validation errors are handled gracefully and provide informative feedback to users.

## Performance Tips

- **Precompile Regular Expressions**: Use `regexp.MustCompile` for better performance compared to `regexp.MatchString`.
- **Minimal Data Copying**: When validating large amounts of data, avoid unnecessary copying of data.
- **Robust Error Messages**: Keep error messages concise and avoid exposing internal logic or sensitive information in error messages.
- **Optimize Validation Logic**: Ensure complex validation logic is efficient, possibly using profiling tools to identify bottlenecks.
