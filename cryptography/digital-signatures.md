---
title: 'Digital Signatures'
description: 'Learn how to generate and verify digital signatures in Go using the crypto package'
date: '2025-03-24'
category: 'Cryptography'
---

Digital signatures are a fundamental cryptographic mechanism providing data authenticity and integrity. Go provides built-in support for digital signatures through its `crypto` package, particularly `crypto/rsa` and `crypto/ecdsa` for RSA and ECDSA protocols respectively.

## Creating and Verifying RSA Signatures

Here's a simple example to generate and verify a digital signature using RSA:

```go
package main

import (
	"crypto"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"fmt"
	"log"	
)

func main() {
	// Generate RSA keys.
	privateKey, err := rsa.GenerateKey(rand.Reader, 2048)
	if err != nil {
		log.Fatal(err)
	}
	publicKey := &privateKey.PublicKey

	// Message to be signed.
	message := []byte("golang crypto signature example")

	// Create a hash of the message.
	hashed := sha256.Sum256(message)

	// Sign the hashed message.
	signature, err := rsa.SignPKCS1v15(rand.Reader, privateKey, crypto.SHA256, hashed[:])
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Signature: %x\n", signature)

	// Verify the signature.
	err = rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, hashed[:], signature)
	if err != nil {
		fmt.Println("Verification failed!")
	} else {
		fmt.Println("Verification successful!")
	}
}
```

## Creating and Verifying ECDSA Signatures

Here's how you can use ECDSA for digital signatures in Go:

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"fmt"
	"log"	
)

func main() {
	// Generate ECDSA keys.
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		log.Fatal(err)
	}
	publicKey := &privateKey.PublicKey

	// Message to be signed.
	message := []byte("golang ecdsa signature example")

	// Create a hash of the message.
	hashed := sha256.Sum256(message)

	// Sign the hashed message.
	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hashed[:])
	if err != nil {
		log.Fatal(err)
	}

	// Verify the signature.
	isValid := ecdsa.Verify(publicKey, hashed[:], r, s)
	if isValid {
		fmt.Println("Verification successful!")
	} else {
		fmt.Println("Verification failed!")
	}
}
```

## Best Practices

- **Key Management**: Ensure that private keys are stored securely and access is restricted to authorized entities only.
- **Hash Function Choice**: Use a cryptographically secure hash function such as SHA-256 or SHA-3 for better security.
- **Error Handling**: Always handle errors gracefully. In cryptography, failing silently can lead to vulnerabilities.

## Common Pitfalls

- **Incorrect Key Usage**: Mixing keys for different cryptographic purposes (e.g., signing and encryption) can compromise security.
- **Ignoring Error Messages**: Ignoring or misinterpreting error messages during verification can lead to false positives in signature validation.
- **Poor Randomness Source**: Use strong random number generators such as `crypto/rand` for generating keys and nonces.

## Performance Tips

- **Use Appropriate Key Sizes**: Balance security and performance by selecting appropriate key sizes. Larger keys offer better security but can be slower.
- **Caching and Reuse**: Cache computed values where possible instead of recalculating them in repeated operations.
- **Profile and Benchmark**: Use Go's testing tools to profile and optimize the performance of your cryptographic operations, particularly for high-volume scenarios.