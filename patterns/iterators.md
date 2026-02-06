---
title: 'Iterators with the iter Package'
description: 'Use Go iterators and the iter package for functional programming patterns and efficient data processing'
date: '2025-08-15'
category: 'Patterns'
---

The `iter` package provides foundational iterator interfaces and helper functions for functional programming patterns. Combined with new iterator support in `slices` and `maps` packages, it enables efficient and expressive data processing.

## Basic Iterator Usage

Work with simple iterators using the `iter.Seq` interface:

```go
package main

import (
	"fmt"
	"iter"
	"slices"
)

// Create a simple iterator that yields numbers.
func numbers(max int) iter.Seq[int] {
	return func(yield func(int) bool) {
		for i := 0; i < max; i++ {
			if !yield(i) {
				return
			}
		}
	}
}

func main() {
	// Use the iterator with range.
	fmt.Println("Numbers 0-4:")
	for num := range numbers(5) {
		fmt.Printf("%d ", num)
	}
	fmt.Println()
	
	// Collect iterator values into a slice.
	nums := slices.Collect(numbers(5))
	fmt.Printf("Collected: %v\n", nums)
}
```

## Slice Iterator Functions

Use new iterator-based functions in the `slices` package:

```go
package main

import (
	"fmt"
	"slices"
	"strconv"
)

func main() {
	data := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	
	// Get all elements with indices as an iterator.
	fmt.Println("All elements with indices:")
	for i, val := range slices.All(data) {
		fmt.Printf("data[%d] = %d ", i, val)
	}
	fmt.Println()
	
	// Get values only.
	fmt.Println("Values only:")
	for val := range slices.Values(data) {
		fmt.Printf("%d ", val)
	}
	fmt.Println()
	
	// Get values backwards.
	fmt.Println("Backwards:")
	for val := range slices.Backward(data) {
		fmt.Printf("%d ", val)
	}
	fmt.Println()
	
	// Chunk into pairs and collect.
	chunked := slices.Collect(
		slices.Chunk(data, 2), // Chunk into pairs.
	)
	fmt.Printf("Chunked data: %v\n", chunked)
	
	// Convert to strings using an iterator
	strings := slices.Collect(func(yield func(string) bool) {
		for _, num := range data {
			if !yield(strconv.Itoa(num)) {
				return
			}
		}
	})
	fmt.Printf("String conversion: %v\n", strings)
}
```

## Map Iterator Functions

Work with map iterators for efficient key-value processing:

```go
package main

import (
	"fmt"
	"maps"
	"slices"
)

func main() {
	userAges := map[string]int{
		"Alice":   30,
		"Bob":     25,
		"Charlie": 35,
		"Diana":   28,
	}
	
	// Get all keys as a slice.
	names := slices.Collect(maps.Keys(userAges))
	slices.Sort(names) // Sort for consistent output.
	fmt.Printf("Names: %v\n", names)
	
	// Get all values as a slice.
	ages := slices.Collect(maps.Values(userAges))
	slices.Sort(ages)
	fmt.Printf("Ages: %v\n", ages)
	
	// Process key-value pairs.
	fmt.Println("User details:")
	for name, age := range maps.All(userAges) {
		fmt.Printf("  %s: %d years old\n", name, age)
	}
	
	// Filter and collect.
	adults := make(map[string]int)
	for name, age := range maps.All(userAges) {
		if age >= 30 {
			adults[name] = age
		}
	}
	fmt.Printf("Adults: %v\n", adults)
}
```

## Key-Value Iterators

Work with `iter.Seq2` for key-value iterations:

```go
package main

import (
	"fmt"
	"iter"	
)

// Create an iterator that yields index-value pairs.
func enumerate[T any](slice []T) iter.Seq2[int, T] {
	return func(yield func(int, T) bool) {
		for i, val := range slice {
			if !yield(i, val) {
				return
			}
		}
	}
}

// Filter key-value pairs.
func filterKV[K, V any](seq iter.Seq2[K, V], predicate func(K, V) bool) iter.Seq2[K, V] {
	return func(yield func(K, V) bool) {
		for k, v := range seq {
			if predicate(k, v) {
				if !yield(k, v) {
					return
				}
			}
		}
	}
}

func main() {
	fruits := []string{"apple", "banana", "cherry", "date", "elderberry"}
	
	// Enumerate with indices.
	fmt.Println("Enumerated fruits:")
	for i, fruit := range enumerate(fruits) {
		fmt.Printf("  %d: %s\n", i, fruit)
	}
	
	// Filter based on index and value.
	longFruits := filterKV(
		enumerate(fruits),
		func(i int, fruit string) bool {
			return len(fruit) > 5 // Fruits with more than 5 characters.
		},
	)
	
	fmt.Println("Long fruits:")
	for i, fruit := range longFruits {
		fmt.Printf("  %d: %s\n", i, fruit)
	}
}
```

## Best Practices

- Use iterators for lazy evaluation to process large datasets efficiently.
- Combine iterator functions to build expressive data processing pipelines.
- Prefer `slices.Collect` to materialize iterator results when needed.
- Use `iter.Seq2` for key-value iterations like enumeration or map processing.

## Common Pitfalls

- Forgetting that iterators are lazy; they don't execute until consumed.
- Not checking the return value of `yield` in custom iterators, preventing early termination.
- Creating infinite iterators without proper termination conditions.

## Performance Tips

- Iterators enable memory-efficient processing of large datasets.
- Chain iterator operations to avoid intermediate slice allocations.
- Use `take` and similar functions to limit processing when appropriate.
- Profile iterator pipelines to ensure they provide the expected performance benefits.
