---
title: 'Understanding Built-in Functions in Go'
description: 'Explore some of the commonly used built-in functions in Go for fundamental operations'
date: '2025-03-24'
category: 'Other topics'
---

Go offers several built-in functions that are fundamental to the language and provide essential functionalities useful in everyday programming tasks. This snippet introduces a few key built-ins like `len`, `cap`, `append`, `copy`, and `panic`.

## Basic Usage of Built-in Functions

### `len` and `cap`

The `len` function returns the number of elements in various data structures, such as arrays, slices, channels, maps, and strings. `cap` returns the capacity, which is applicable to slices, arrays, and channels.

```go
package main

import "fmt"

func main() {
	s := []int{1, 2, 3, 4, 5}

	fmt.Println("Length:", len(s)) // Output: 5
	fmt.Println("Capacity:", cap(s)) // Output: 5
}
```

### `append`

The `append` function is used to add elements to a slice, dynamically increasing its length.

```go
package main

import "fmt"

func main() {
	s := []int{1, 2, 3}
	s = append(s, 4, 5)

	fmt.Println(s) // Output: [1 2 3 4 5]
}
```

### `copy`

The `copy` function copies elements from one slice into another, returning the number of elements copied.

```go
package main

import "fmt"

func main() {
	src := []int{1, 2, 3}
	dst := make([]int, len(src))

	n := copy(dst, src)

	fmt.Println("Copied:", n) // Output: 3
	fmt.Println("Destination:", dst) // Output: [1 2 3]
}
```

### `panic`

The `panic` function is used to handle unexpected errors by stopping the execution of the program and beginning a panicking sequence.

```go
package main

import (
	"os"
	"log"
)

func main() {
	f, err := os.Open("nonexistent.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()
}
```

## Best Practices

- Use `len` and `cap` to manage the size and capacity of slices effectively.
- When appending to slices, be mindful of the underlying array capacity and be prepared for it to double if needed.
- Use `copy` for efficiently cloning slices when a shallow copy suffices.
- Reserve `panic` for unrecoverable errors and program conditions that should not occur during normal execution.

## Common Pitfalls

- Misunderstanding the difference between `len` and `cap` can lead to inefficient slice management.
- Modifying slices without considering their append behavior can lead to unexpected data overwrites in shared backing arrays.
- Excessive reliance on `panic` for error handling can make the codebase fragile and difficult to maintain.

## Performance Tips

- Minimize reallocations by pre-sizing slices where possible using `make` with an appropriate capacity.
- Use `copy` instead of manual loops for slice duplication for better performance.
- Avoid excessive use of `append` in critical paths; consider pre-allocated buffers or builders for large data aggregations.
- Use `panic` sparingly in core libraries to maintain a robust operational codebase.
