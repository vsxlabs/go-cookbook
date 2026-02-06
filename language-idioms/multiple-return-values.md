---
title: 'Multiple Return Values'
description: 'Learn how to utilize multiple return values in Go functions for cleaner and more informative code.'
date: '2025-03-24'
category: 'Tools'
---

Go, as a language, offers a powerful feature: multiple return values from functions. This feature can significantly enhance the usability and clarity of your functions, allowing them to return detailed results and errors in a concise manner.

## Basic Example of Multiple Return Values

Here's how you can use multiple return values in Go functions:

```go
package main

import (
	"errors"
	"fmt"
)

// Divide divides two numbers and returns the result and an error if any.
func Divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("cannot divide by zero")
	}
	return a / b, nil
}

func main() {
	result, err := Divide(10, 2)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result) // Output: Result: 5
	}
}
```

## Using Named Return Values

Go allows you to name the return values, which can be handy for documentation purposes:

```go
package main

import (
	"fmt"
)

// CompareAndSwap swaps values if the first is greater than the second.
func CompareAndSwap(a, b int) (newA int, newB int, wasSwapped bool) {
	if a > b {
		return b, a, true
	}
	return a, b, false
}

func main() {
	a, b, swapped := CompareAndSwap(10, 5)
	if swapped {
		fmt.Println("Values were swapped:", a, b)
	} else {
		fmt.Println("No swap needed:", a, b)
	}
}
```

## Multiple Return Values for Simplicity

In Go, it's common to use the multiple return value feature to return both a value and an error:

```go
package main

import (
	"errors"
	"fmt"
)

func GetElement(slice []int, index int) (element int, err error) {
	if index < 0 || index >= len(slice) {
		return 0, errors.New("index out of bounds")
	}
	return slice[index], nil
}

func main() {
	numbers := []int{10, 20, 30, 40}
	element, err := GetElement(numbers, 2)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Element at index 2:", element)
	}
}
```

## Best Practices

- Use multiple returns to provide error messages alongside results. This makes error handling in your programs more robust and informative.
- Consider naming your return values when they improve comprehension, especially for functions returning similar value types.
- Maintain consistency in error returning. A nil error should always accompany a valid result, while an error should be returned with the function's zero value for other returns.

## Common Pitfalls

- Forgetting to check the second return value for errors could lead to subtle bugs.
- Overusing named returns can confuse readers and misuse memory, especially if they don't add clarity.
- Not handling nil values where expected or not defaulting values when errors occur.

## Performance Tips

- Named return values can sometimes consume more memory if not used prudently, especially if large structures are involved.
- Avoid unnecessary computations after an error is detected; always check for errors immediately.
- Consider inline short forms to reduce unnecessary assignments when only capturing errors from some operations, e.g., `_ , err := func() ...`.