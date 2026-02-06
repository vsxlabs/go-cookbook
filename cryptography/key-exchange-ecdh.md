---
title: 'Elliptic Curve Diffie-Hellman Key Exchange'
description: 'Implement secure key exchange using the crypto/ecdh package for establishing shared secrets'
date: '2025-08-15'
category: 'Cryptography'
---

The `crypto/ecdh` package provides Elliptic Curve Diffie-Hellman (ECDH) key exchange functionality, enabling two parties to establish a shared secret over an insecure channel for secure communication.

## Basic ECDH Key Exchange

Implement a simple key exchange between two parties:

```go
package main

import (
	"crypto/ecdh"
	"crypto/rand"
	"fmt"
	"log"	
)

func main() {
	// Choose a curve - P256 is widely supported and secure.
	curve := ecdh.P256()
	
	// Alice generates her key pair.
	alicePrivate, err := curve.GenerateKey(rand.Reader)
	if err != nil {
		log.Fatal(err)
	}
	alicePublic := alicePrivate.PublicKey()
	
	// Bob generates his key pair.
	bobPrivate, err := curve.GenerateKey(rand.Reader)
	if err != nil {
		log.Fatal(err)
	}
	bobPublic := bobPrivate.PublicKey()
	
	// Alice computes the shared secret using Bob's public key.
	aliceSecret, err := alicePrivate.ECDH(bobPublic)
	if err != nil {
		log.Fatal(err)
	}
	
	// Bob computes the shared secret using Alice's public key.
	bobSecret, err := bobPrivate.ECDH(alicePublic)
	if err != nil {
		log.Fatal(err)
	}
	
	// The shared secrets should be identical.
	fmt.Printf("Alice's shared secret: %x\n", aliceSecret)
	fmt.Printf("Bob's shared secret:   %x\n", bobSecret)
	fmt.Printf("Secrets match: %t\n", string(aliceSecret) == string(bobSecret))
}
```

## Best Practices

- Always use a cryptographically secure random number generator for key generation.
- Choose appropriate curves based on security requirements: P-256 for most applications, P-384 or P-521 for higher security needs.
- Derive encryption keys from the shared secret using a proper key derivation function.
- Implement proper key validation to prevent invalid point attacks.

## Common Pitfalls

- Using the shared secret directly as an encryption key without key derivation.
- Not validating peer public keys, which can lead to security vulnerabilities.
- Reusing ephemeral keys across multiple exchanges.
- Not handling curve point validation properly.

## Performance Tips

- P-256 offers the best performance-to-security ratio for most applications.
- Cache derived keys when possible to avoid repeated expensive operations.
- Consider using hardware acceleration when available for elliptic curve operations.
- Pre-generate key pairs in advance for high-throughput applications when feasible.
