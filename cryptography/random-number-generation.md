---
title: 'Random Number Generation'
description: 'Learn how to generate random numbers in Go using the crypto and math packages for cryptographic and non-cryptographic purposes'
date: '2025-03-24'
category: 'Cryptography'
---

In Go, random number generation can be split into two main categories: non-cryptographic and cryptographic. For non-cryptographic purposes, the `math/rand` package is commonly used. For cryptographic purposes, the `crypto/rand` package ensures more secure random number generation.

## Non-Cryptographic Random Number Generation

Use the `math/rand` package for general-purpose random number generation.

### Example

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().UnixNano()) // Seed the random number generator
	fmt.Println(rand.Intn(73))       // Random integer between 0 and 72.
	fmt.Println(rand.Float64())      // Random float between 0.0 and 1.0.
}
```

## Cryptographic Random Number Generation

For secure applications, use the `crypto/rand` package.

### Example

```go
package main

import (
	"crypto/rand"
	"fmt"
	"log"
	"math/big"	
)

func main() {
	n, err := rand.Int(rand.Reader, big.NewInt(73)) // Random integer between 0 and 72 (inclusive).
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(n)
}
```

## Generating Random Bytes

Random bytes are often required for cryptographic keys or tokens.

```go
package main

import (
	"crypto/rand"
	"fmt"
)

func main() {
	bytes := make([]byte, 32) // Generate 32 random bytes.
	if _, err := rand.Read(bytes); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Random Bytes: %x\n", bytes)
}
```

## Modern Random Number Generation with math/rand/v2

As of Go 1.22, `math/rand/v2` provides significant improvements over the original `math/rand` package, including better pseudo-random number generation algorithms, automatic seeding that eliminates the need for manual seeding, and enhanced concurrency support with per-goroutine generators for better performance in multi-threaded applications.

## Best Practices

- **Cryptographic random numbers:** Always use `crypto/rand` for cryptographic needs such as generating keys, nonces, or tokens.
- **Seeding:** Seed `math/rand` using the current time or a unique value for different sequences in different runs. Note that `math/rand/v2` provides automatic seeding and improved algorithms.
- **Error Handling:** Handle errors appropriately in cryptographic operations; do not ignore them.

## Common Pitfalls

- **Re-seeding `math/rand`:** Re-seeding `math/rand` multiple times can result in poor random sequences and predictability.
- **Ignoring crypto errors:** Failing to check for errors in `crypto/rand` can lead to undetected secure random number generation failures.
- **Inappropriate use:** Using `math/rand` where cryptographic security is required can lead to vulnerabilities.

## Performance Tips

- **Efficiency:** Use `math/rand` or `math/rand/v2` when cryptographic security is not needed for better performance as they are less computationally intensive.
- **Buffering:** If you need many random numbers at once, consider generating them in a batch and storing them in a buffer for repeated access.
- **Concurrency:** Leverage concurrency when generating random numbers in large-scale applications to optimize performance. `math/rand/v2` provides better concurrency support.