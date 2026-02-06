---
title: 'Value Interning with the unique Package'
description: 'Reduce memory usage by canonicalizing comparable values using the unique package'
date: '2025-08-15'
category: 'Memory Management'
---

The `unique` package provides value interning functionality, allowing you to canonicalize comparable values to reduce memory usage. When you intern a value, you get a handle that ensures only one copy of that value exists in memory.

## Basic Value Interning

Use `unique.Make` to intern comparable values and get handles for efficient comparison:

```go
package main

import (
	"fmt"
	"unique"
)

func main() {
	// Intern string values
	str1 := unique.Make("hello world")
	str2 := unique.Make("hello world")
	str3 := unique.Make("different string")
	
	// Handles can be compared cheaply by pointer
	fmt.Printf("str1 == str2: %t (same interned value)\n", str1 == str2)
	fmt.Printf("str1 == str3: %t (different values)\n", str1 == str3)
	
	// Get the original value back
	fmt.Printf("Value from handle: %s\n", str1.Value())
}
```

## Interning Complex Structures

Intern custom structs to reduce memory usage for duplicate data:

```go
package main

import (
	"fmt"
	"unique"
)

type Person struct {
	Name string
	Age  int
	City string
}

func main() {
	// Create multiple instances of the same person data.
	person1 := unique.Make(Person{Name: "Alice", Age: 30, City: "New York"})
	person2 := unique.Make(Person{Name: "Alice", Age: 30, City: "New York"})
	person3 := unique.Make(Person{Name: "Bob", Age: 25, City: "Boston"})
	
	// Memory-efficient comparison.
	if person1 == person2 {
		fmt.Println("Same person data - only one copy in memory")
	}
	
	// Different data results in different handles.
	if person1 != person3 {
		fmt.Println("Different person data")
	}
	
	// Access the original struct.
	fmt.Printf("Person 1: %+v\n", person1.Value())
}
```

## Building a Deduplication Cache

Use interning to build memory-efficient caches that automatically deduplicate values:

```go
package main

import (
	"fmt"
	"unique"
)

// DeduplicatedCache stores unique handles to avoid duplicate data.
type DeduplicatedCache[K comparable, V comparable] struct {
	data map[K]unique.Handle[V]
}

func NewDeduplicatedCache[K comparable, V comparable]() *DeduplicatedCache[K, V] {
	return &DeduplicatedCache[K, V]{
		data: make(map[K]unique.Handle[V]),
	}
}

func (c *DeduplicatedCache[K, V]) Set(key K, value V) {
	c.data[key] = unique.Make(value)
}

func (c *DeduplicatedCache[K, V]) Get(key K) (V, bool) {
	handle, exists := c.data[key]
	if !exists {
		var zero V
		return zero, false
	}
	return handle.Value(), true
}

func (c *DeduplicatedCache[K, V]) GetHandle(key K) (unique.Handle[V], bool) {
	handle, exists := c.data[key]
	return handle, exists
}

func main() {
	cache := NewDeduplicatedCache[int, string]()
	
	// Store the same value multiple times.
	cache.Set(1, "common value")
	cache.Set(2, "common value")
	cache.Set(3, "common value")
	cache.Set(4, "unique value")
	
	// Get handles to compare.
	h1, _ := cache.GetHandle(1)
	h2, _ := cache.GetHandle(2)
	h3, _ := cache.GetHandle(3)
	h4, _ := cache.GetHandle(4)
	
	// Same values share the same interned instance.
	fmt.Printf("h1 == h2: %t (same interned value)\n", h1 == h2)
	fmt.Printf("h2 == h3: %t (same interned value)\n", h2 == h3)
	fmt.Printf("h1 == h4: %t (different value)\n", h1 == h4)
	
	// Retrieve values normally.
	val1, _ := cache.Get(1)
	fmt.Printf("Retrieved value: %s\n", val1)
}
```

## Memory-Efficient String Processing

Use interning for processing large datasets with repeated string values:

```go
package main

import (
	"fmt"
	"unique"
)

type LogEntry struct {
	Level     unique.Handle[string]
	Service   unique.Handle[string]
	Message   string
	Timestamp int64
}

func processLogEntries(rawLogs []map[string]any) []LogEntry {
	var entries []LogEntry
	
	for _, raw := range rawLogs {
		entry := LogEntry{
			Level:     unique.Make(raw["level"].(string)),
			Service:   unique.Make(raw["service"].(string)),
			Message:   raw["message"].(string),
			Timestamp: raw["timestamp"].(int64),
		}
		entries = append(entries, entry)
	}
	
	return entries
}

func main() {
	// Simulate log data with repeated level and service values.
	rawLogs := []map[string]any{
		{"level": "ERROR", "service": "user-service", "message": "Login failed", "timestamp": int64(1640995200)},
		{"level": "INFO", "service": "user-service", "message": "User created", "timestamp": int64(1640995201)},
		{"level": "ERROR", "service": "payment-service", "message": "Payment failed", "timestamp": int64(1640995202)},
		{"level": "INFO", "service": "user-service", "message": "Password updated", "timestamp": int64(1640995203)},
		{"level": "ERROR", "service": "user-service", "message": "Invalid token", "timestamp": int64(1640995204)},
	}
	
	entries := processLogEntries(rawLogs)
	
	// Count unique level and service combinations.
	levelCounts := make(map[unique.Handle[string]]int)
	serviceCounts := make(map[unique.Handle[string]]int)
	
	for _, entry := range entries {
		levelCounts[entry.Level]++
		serviceCounts[entry.Service]++
	}
	
	fmt.Printf("Unique log levels: %d\n", len(levelCounts))
	fmt.Printf("Unique services: %d\n", len(serviceCounts))
	
	// Display counts.
	for level, count := range levelCounts {
		fmt.Printf("Level %s: %d entries\n", level.Value(), count)
	}
}
```

## Best Practices

- Use interning for values that are frequently duplicated across your application.
- Intern configuration data, enum-like strings, and repeated structured data.
- Compare handles directly for fast equality checks instead of comparing values.
- Be aware that interned values are retained until no handles reference them.

## Common Pitfalls

- Over-interning small or rarely duplicated values can increase memory usage.
- Forgetting that handles need to be dereferenced with `.Value()` to access the original data.
- Not considering the cost of interning versus the memory savings for your specific use case.

## Performance Tips

- Interning provides both memory savings and faster equality comparisons.
- Use handles for map keys when you need fast lookups with interned values.
- Profile memory usage before and after interning to measure the impact.
- Consider interning at data ingestion boundaries for maximum benefit across the application.
