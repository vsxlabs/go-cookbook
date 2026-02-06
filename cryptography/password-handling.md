---
title: 'Password Handling with Cryptography'
description: 'Learn secure password handling in Go using bcrypt for hashing and validation'
date: '2025-03-24'
category: 'Cryptography'
---

Handling passwords securely is vital for safeguarding user data. Go provides a robust package, `golang.org/x/crypto/bcrypt`, for hashing and verifying passwords, which helps prevent exposure of plain-text passwords and supports secure authentication practices.

## Password Hashing

`bcrypt` is the de-facto standard library in Go for password hashing due to its strength against brute force attacks. Here's how you can use `bcrypt` to hash a password:

```go
package main

import (
	"fmt"
	"golang.org/x/crypto/bcrypt"
)

func main() {
	password := "golangSecure73"

	// Generate hashed password.
	hashedPassword, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Hashed Password:", string(hashedPassword))
}
```

## Password Verification

To verify a password, compare the hashed password with the plain-text password using `bcrypt.CompareHashAndPassword`:

```go
package main

import (
	"fmt"
	"golang.org/x/crypto/bcrypt"
)

func main() {
	storedHash := "$2a$10$6npNamP1BNFyEWAZHQvtsOgrBiAdCHpH2z.bxHV.xT1.ntIWx8hCC"

	actualPassword := "golangSecure73"

	// Check password.
	err := bcrypt.CompareHashAndPassword([]byte(storedHash), []byte(actualPassword))
	if err != nil {
		fmt.Println("Invalid password!")
	} else {
		fmt.Println("Password verified successfully!")
	}
}
```

## Best Practices

- **Use Recommended Cost:** Use `bcrypt.DefaultCost` unless you have evaluated your system's capacity for different costs. Higher cost increases security but also CPU usage.
- **Salt Automatically:** `bcrypt` automatically salts passwords, so you don't need to handle it manually.
- **Secure Storage:** Store only the hashed password, never the plain text or the salt.
- **Upgrade Policies:** Regularly review and increase the cost parameter as computational power grows.

## Common Pitfalls

- **Cost Mismanagement:** Setting the cost too high may lead to performance issues, while setting it too low makes it easier to brute-force.
- **Neglecting Error Handling:** Failing to properly handle errors from `bcrypt` functions can lead to problems in password validation workflows.
- **Insecure Random Sources:** Ensure that the library implementation uses a secure source of randomness; if handling custom implementations, verify secure randomness.

## Performance Tips

- **Benchmarking Costs:** Benchmark your selected cost against your system to determine an acceptable balance between security and performance.
- **Batch Processing:** In high-throughput systems, consider processing password checks concurrently to reduce latency.
- **Profile Your Application:** Frequently profile your application to identify any deviations in performance, particularly when increasing cost factors or running in resource-constrained environments.