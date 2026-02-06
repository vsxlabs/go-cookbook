---
title: 'Weak References for Memory-Efficient Caches'
description: 'Use the weak package to build memory-efficient caches and canonicalized value maps'
date: '2025-08-15'
category: 'Memory Management'
---

The `weak` package provides weak references that allow objects to be garbage collected even when referenced by cache structures, enabling memory-efficient caches and canonicalized value maps without memory leaks.

## Basic Weak Reference Usage

Create weak references that don't prevent garbage collection:

```go
package main

import (
	"fmt"
	"runtime"
	"weak"
)

func main() {
	// Create an object.
	obj := &struct {
		data string
	}{data: "important data"}
	
	// Create a weak reference to the object.
	weakRef := weak.Make(obj)
	
	// The weak reference doesn't prevent GC.
	fmt.Printf("Object via weak ref: %+v\n", weakRef.Value())
	
	// Clear the strong reference.
	obj = nil
	
	// Force garbage collection.
	runtime.GC()
	runtime.GC() // May need multiple calls.
	
	// The weak reference may now return nil.
	if weakRef.Value() == nil {
		fmt.Println("Object was garbage collected")
	} else {
		fmt.Println("Object still exists")
	}
}
```

## Memory-Efficient Cache Implementation

Build a cache using weak references that automatically cleans up unused entries:

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"weak"
)

// WeakCache implements a cache that allows entries to be garbage collected.
type WeakCache[K comparable, V any] struct {
	mu    sync.RWMutex
	cache map[K]*weak.Pointer[V]
}

func NewWeakCache[K comparable, V any]() *WeakCache[K, V] {
	return &WeakCache[K, V]{
		cache: make(map[K]*weak.Pointer[V]),
	}
}

func (c *WeakCache[K, V]) Set(key K, value *V) {
	c.mu.Lock()
	defer c.mu.Unlock()
	weakPtr := weak.Make(value)
	c.cache[key] = &weakPtr
}

func (c *WeakCache[K, V]) Get(key K) *V {
	c.mu.RLock()
	weakPtr, exists := c.cache[key]
	c.mu.RUnlock()
	
	if !exists {
		return nil
	}
	
	value := weakPtr.Value()
	if value == nil {
		// Clean up dead reference.
		c.mu.Lock()
		delete(c.cache, key)
		c.mu.Unlock()
	}
	
	return value
}

func (c *WeakCache[K, V]) Cleanup() {
	c.mu.Lock()
	defer c.mu.Unlock()
	
	for key, weakPtr := range c.cache {
		if weakPtr.Value() == nil {
			delete(c.cache, key)
		}
	}
}

func (c *WeakCache[K, V]) Size() int {
	c.mu.RLock()
	defer c.mu.RUnlock()
	return len(c.cache)
}

func demonstrateWeakCache() {
	cache := NewWeakCache[string, string]()
	
	// Add some values.
	val1 := "expensive computation result 1"
	val2 := "expensive computation result 2"
	val3 := "expensive computation result 3"
	
	cache.Set("key1", &val1)
	cache.Set("key2", &val2)
	cache.Set("key3", &val3)
	
	fmt.Printf("Cache size after adding: %d\n", cache.Size())
	
	// Clear some strong references.
	val2 = ""
	val3 = ""
	
	// Force garbage collection.
	runtime.GC()
	runtime.GC()
	
	// Check what's still available.
	if result := cache.Get("key1"); result != nil {
		fmt.Printf("key1 still available: %s\n", *result)
	}
	
	if result := cache.Get("key2"); result == nil {
		fmt.Println("key2 was garbage collected")
	}
	
	if result := cache.Get("key3"); result == nil {
		fmt.Println("key3 was garbage collected")
	}
	
	fmt.Printf("Cache size after GC: %d\n", cache.Size())
}
```

## Best Practices

- Use weak references for caches and maps where automatic cleanup is desired.
- Combine with periodic cleanup routines for immediate memory reclamation.
- Be aware that weak references may return nil at any time after the last strong reference is removed.
- Test garbage collection behavior in your specific use case.

## Common Pitfalls

- Forgetting that weak references can become nil between checks.
- Not implementing proper cleanup of dead weak references from data structures.
- Relying on immediate cleanup without forcing garbage collection in tests.

## Performance Tips

- Weak references add minimal overhead compared to memory leaks from strong references.
- Implement lazy cleanup to avoid performance impact from frequent dead reference scanning.
- Consider using weak references in combination with LRU eviction for hybrid cache strategies.
- Monitor cache hit rates to ensure weak reference cleanup isn't too aggressive.
