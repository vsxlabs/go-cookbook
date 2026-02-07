---
title: 'Go Slices'
description: 'An essential guide to using slices in Go, covering slice creation, manipulation, and performance tips'
date: '2025-03-24'
category: 'Tools'
---

Slices are a powerful feature in Go, providing a more flexible and convenient way to work with sequences of elements compared to arrays. This guide introduces Go slices, demonstrates basic and advanced operations, and offers best practices and tips for maximizing performance.

## Creating and Initializing Slices

Slices can be created using the `make` function or through slicing an existing array or slice.

### Using the `make` Function

```go
package main

import "fmt"

func main() {
	// Create a slice with length 3 and capacity 10.
	slice := make([]int, 3, 10)
	fmt.Println(slice) // Output: [0 0 0].
}
```

### Slicing an Array

```go
package main

import "fmt"

func main() {	
	arr := [5]int{1, 2, 3, 4, 5}
	// Create a slice from the array.
	slice := arr[1:4]
	fmt.Println(slice) // Output: [2 3 4].
}
```

## Appending to Slices

The `append` function is used to add elements to a slice.

```go
package main

import "fmt"

func main() {
	// Initial slice.
	slice := []int{1, 2, 3}
	// Append one element.
	slice = append(slice, 4)
	// Append multiple elements.
	slice = append(slice, 5, 6, 7)
	fmt.Println(slice) // Output: [1 2 3 4 5 6 7].
}
```

## Copying Slices

You can copy elements from one slice to another using the `copy` function.

```go
package main

import "fmt"

func main() {
	source := []int{1, 2, 3, 4}
	destination := make([]int, len(source))
	copy(destination, source)
	fmt.Println(destination) // Output: [1 2 3 4].
}
```

## Modifying Slices

Slices support range to perform iteration, allowing easy modification through assignment.

```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3, 4, 5}
	// Double each element.
	for i := range slice {
		slice[i] *= 2
	}
	fmt.Println(slice) // Output: [2 4 6 8 10].
}
```

## Best Practices

- Use `append` carefully, as it might lead to allocation of a new underlying array if the capacity is exceeded.

## Common Pitfalls

- Misunderstanding slices as arrays — slices are reference types and share the backing array.
- Forgetting that modifying a slice might affect all slices pointing to the same array.
- Using an uninitialized slice (`nil` slice) without checking as it doesn't automatically allocate storage.

## Slice Utility Functions

The `slices` package provides many utility functions for common slice operations:

```go
package main

import (
	"fmt"
	"slices"
)

func main() {
	numbers := []int{3, 1, 4, 1, 5, 9, 2, 6}

	// Find elements
	fmt.Printf("Contains 5: %t\n", slices.Contains(numbers, 5))
	fmt.Printf("Index of 4: %d\n", slices.Index(numbers, 4))

	// Sorting and binary search
	sortedNumbers := slices.Clone(numbers)
	slices.Sort(sortedNumbers)
	fmt.Printf("Sorted: %v\n", sortedNumbers)
	binarySearchRes, _ := slices.BinarySearch(sortedNumbers, 5)
	fmt.Printf("Binary search for 5: %d\n", binarySearchRes)

	// Insert and delete
	inserted := slices.Insert(numbers, 2, 99, 100)
	fmt.Printf("After insert: %v\n", inserted)

	deleted := slices.Delete(numbers, 1, 3)
	fmt.Printf("After delete: %v\n", deleted)

	// Comparison and equality
	other := []int{3, 1, 4, 1, 5, 9, 2, 6}
	fmt.Printf("Equal to other: %t\n", slices.Equal(numbers, other))

	// Reverse
	reversed := slices.Clone(numbers)
	slices.Reverse(reversed)
	fmt.Printf("Reversed: %v\n", reversed)
}
```

## Performance Tips

- Pre-allocate slices using `make` function to avoid multiple allocations.
- Be mindful of large memory allocations during `append` operations; use `cap()` to optimize capacity size.
- Passing a slice by value only copies the slice header (pointer, length, capacity), not the underlying elements. However, be aware that modifications to elements will affect all copies since they share the same backing array.
- Use slicing operations carefully to avoid inadvertent high-memory usage by sharing a large backing array.
- Use `slices` package utilities for common operations like sorting, searching, and manipulation for better performance and correctness.