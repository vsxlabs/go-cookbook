---
title: 'Go Maps'
description: 'Learn how to create, use, and manipulate maps in Go, including best practices and performance tips.'
date: '2025-03-24'
category: 'Tools'
---

Maps in Go provide an excellent way to store and efficiently retrieve key-value pairs. This guide will walk you through the basics of using maps, along with some advanced usage scenarios.

## Basic Map Usage

Here's a simple example demonstrating how to create and use maps in Go:

```go
package main

import (
	"fmt"
)

func main() {
	// Create a map with string keys and int values.
	ages := map[string]int{
		"Alice": 15,
		"Bob":   20,
	}

	// Add a key-value element.
	ages["Chris"] = 25

	// Retrieve and print a value.
	if age, exists := ages["Alice"]; exists {
		fmt.Printf("Alice is %d years old.\n", age)
	} else {
		fmt.Println("Age for Alice not found.")
	}
	
	// Remove a key-value element.
	delete(ages, "Bob")

	// Print the map.
	fmt.Println("Updated ages:", ages)
}
```

## Checking for Key Existence and Iteration

```go
package main

import (
	"fmt"
)

func main() {
	// Initialize a map.
	cities := map[string]string{
		"NY": "New York",
		"LA": "Los Angeles",
	}

	// Check if a key exists.
	if city, found := cities["NY"]; found {
		fmt.Printf("The abbreviation 'NY' stands for %s.\n", city)
	} else {
		fmt.Println("Key not found.")
	}

	// Iterate over a map.
	for abbrev, city := range cities {
		fmt.Printf("%s -> %s\n", abbrev, city)
	}
}
```

## Nested Maps

Using nested maps can be handy in situations where hierarchical data is needed:

```go
package main

import (
	"fmt"
)

func main() {
	departments := make(map[string]map[string]int)

	// Add nested maps.
	departments["HR"] = map[string]int{
		"Alice": 35,
		"Bob":   40,
	}

	// Retrieve a nested map.
	if hrDept, exists := departments["HR"]; exists {
		for name, age := range hrDept {
			fmt.Printf("%s in HR is %d years old.\n", name, age)
		}
	}
}
```

## Best Practices

- For multi-threaded programs, consider using `sync.Map` for concurrent access.
- Always check for existence using the `value, ok := map[key]` form to prevent unexpected zero values.
- Remember that maps are reference types; changes to a map in one part of your program will affect all references to it.

## Common Pitfalls

- Be cautious of the zero value behavior; accessing a non-existent key returns the zero value for the map's value type.
- Concurrent map access from multiple goroutines (reading while another writes, or multiple writers) causes runtime panics. Use synchronization (e.g., `sync.RWMutex`) or `sync.Map` for concurrent access. Note: deleting entries during iteration in a single goroutine is safe.
- Forgetting that maps are not deep-copied automatically—sharing a map reference can lead to unintended data changes.

## Performance Tips

- Minimize map growth by using `make` with a hint for how many elements you expect to store, e.g., `make(map[string]int, expectedSize)`.
- Maps are generally not optimized for high write/read concurrency; for high-concurrency applications, consider other data structures like sync.RWMutex or `sync.Map`.
- Use pre-allocated slices when you need to frequently iterate over map values and optimize lookups.

By following these practices and being aware of common pitfalls, you’ll be able to effectively use maps in your Go programs to handle key-value pairs seamlessly.
