---
title: 'Encryption and Decryption'
description: 'Learn how to perform encryption and decryption in Go.'
date: '2025-03-24'
category: 'Cryptography'
---

Cryptography in Go can be performed using various packages that provide implementations of common encryption algorithms. This snippet focuses on basic encryption and decryption using the `crypto/cipher` package.

## Basic Encryption and Decryption using AES

The `crypto/cipher` package in Go supports a variety of encryption algorithms, of which AES (Advanced Encryption Standard) is widely used. Below is a simple example of AES encryption and decryption in Go.

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/hex"
	"fmt"
	"io"
	"log"	
)

func encrypt(plaintext string, key []byte) (string, error) {
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	ciphertext := make([]byte, aes.BlockSize+len(plaintext))
	iv := ciphertext[:aes.BlockSize]
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return "", err
	}

	stream := cipher.NewCFBEncrypter(block, iv)
	stream.XORKeyStream(ciphertext[aes.BlockSize:], []byte(plaintext))

	return fmt.Sprintf("%x", ciphertext), nil
}

func decrypt(encryptedHex string, key []byte) (string, error) {
	ciphertext, err := hex.DecodeString(encryptedHex)
	if err != nil {
		return "", fmt.Errorf("invalid hex string: %w", err)
	}
	
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	if len(ciphertext) < aes.BlockSize {
		return "", fmt.Errorf("ciphertext too short")
	}

	iv := ciphertext[:aes.BlockSize]
	ciphertext = ciphertext[aes.BlockSize:]

	stream := cipher.NewCFBDecrypter(block, iv)
	stream.XORKeyStream(ciphertext, ciphertext)

	return string(ciphertext), nil
}

func main() {
	// WARNING: Hardcoded keys are insecure. In production, use securely generated keys
	// from environment variables, configuration files, or a key management service (KMS).
	key := []byte("examplekey123456") // 16 bytes key for AES-128.
	plaintext := "Hello, Go Cookbook!"

	encrypted, err := encrypt(plaintext, key)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Encrypted: %s\n", encrypted)

	decrypted, err := decrypt(encrypted, key)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Decrypted: %s\n", decrypted)
}
```

## Best Practices

- Use a secure key exchange mechanism to distribute encryption keys.
- Ensure keys are stored securely and rotated regularly.
- Validate the encryption scheme's authenticity (e.g., using HMAC).
- Consider using AEAD (Authenticated Encryption with Associated Data) schemes like GCM.

## Common Pitfalls

- Using weak or hardcoded encryption keys.
- Neglecting proper error handling could lead to security issues.
- Forgetting to initialize the IV (Initialization Vector) securely.
- Not checking for common vulnerabilities (e.g., padding oracle attack).

## Performance Tips

- Use streaming modes (like CFB) for large data, avoiding loading it entirely in memory.
- Benchmark and profile your code to identify opportunities for optimization.
- Consider hardware-accelerated encryption libraries for performance-critical applications.