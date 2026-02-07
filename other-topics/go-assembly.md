---
title: 'Go Assembly'
description: 'Learn how to work with assembly code in Go by writing .s files and using the Go toolchain'
date: '2025-03-24'
category: 'Other topics'
---

The Go programming language supports writing functions in assembly, which can be useful for performance-critical sections of code. Go assembly is written in `.s` files and compiled by the Go toolchain (`go tool asm`). It allows you to directly manipulate registers and perform low-level operations, which can yield significant performance improvements for specific algorithms.

## Writing a Simple Assembly Function

This example demonstrates how to write a simple assembly function in Go that adds two integers.

### Go Code

```go
package main

import "fmt"

// Add adds two integers using an assembly implementation.
func Add(a, b int) int

func main() {
	result := Add(3, 4)
	fmt.Println("Result:", result)
}
```

### Assembly Code (add.s)

```assembly
TEXT ·Add(SB),0,$0
    MOVQ a+0(FP), AX // Load 'a' from the first argument
    ADDQ b+8(FP), AX // Add 'b' to 'AX' register
    MOVQ AX, ret+16(FP) // Store result in return location
    RET
```

## Best Practices

- Use assembly for performance-critical code where Go’s high-level abstractions do not offer sufficient performance.
- Use comments in assembly code to describe operations, as they can be cryptic.

## Common Pitfalls

- Errors in assembly code can lead to crashes or undefined behavior, so extensive testing is required.
- Assembly code is often architecture-specific; ensure compatibility across architectures or provide architecture-specific implementations.
- Debugging assembly can be complex. Use minimal assembly and fall back to Go for less critical operations.

## Performance Tips

- Profile the Go code to identify bottlenecks before optimizing with assembly.
- Keep assembly functions small and focused; interface them with Go functions for larger algorithms.
- Leverage Go’s escape analysis to minimize unnecessary allocations when working with assembly.