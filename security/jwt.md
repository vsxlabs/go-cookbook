---
title: 'Working with JWT in Go'
description: 'Learn how to generate, parse, and validate JSON Web Tokens (JWTs) in Go using the go-jose library'
date: '2025-03-24'
category: 'Security'
---

JSON Web Tokens (JWTs) are used extensively for securing APIs and managing user sessions. In Go, the `go-jose` library is a popular choice for handling JWTs due to its comprehensive support for JSON Web Encryption (JWE) and JSON Web Signature (JWS).

## Generating a JWT

Here's a basic example illustrating how to generate a JWT:

```go
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"github.com/go-jose/go-jose/v4"
	"github.com/go-jose/go-jose/v4/jwt"
)

func main() {
	// Load the secret key from environment variable.
	secretKey := os.Getenv("JWT_SECRET_KEY")
	if secretKey == "" {
		log.Fatal("JWT_SECRET_KEY environment variable is required")
	}
	key := []byte(secretKey)

	// Create a signer.
	signer, err := jose.NewSigner(jose.SigningKey{Algorithm: jose.HS256, Key: key}, nil)
	if err != nil {
		log.Fatalf("Failed to create signer: %v", err)
	}

	// Define claims.
	cl := jwt.Claims{
		Issuer:    "example.com",
		Audience:  jwt.Audience{"example-audience"},
		Expiry:    jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
		IssuedAt:  jwt.NewNumericDate(time.Now()),
	}

	// Create and sign the JWT.
	raw, err := jwt.Signed(signer).Claims(cl).Serialize()
	if err != nil {
		log.Fatalf("Failed to create JWT: %v", err)
	}

	fmt.Printf("JWT: %s\n", raw)
}
```

## Parsing and Validating a JWT

Below is an example of how to parse and validate a JWT:

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/go-jose/go-jose/v4"
	"github.com/go-jose/go-jose/v4/jwt"
)

func main() {
	// Simulate a JWT string received from somewhere.
	rawJWT := "your.jwt.string.here"

	// Load the secret key from environment variable.
	secretKey := os.Getenv("JWT_SECRET_KEY")
	if secretKey == "" {
		log.Fatal("JWT_SECRET_KEY environment variable is required")
	}
	key := []byte(secretKey)

	// Parse the JWT.
	token, err := jwt.ParseSigned(rawJWT, []jose.SignatureAlgorithm{jose.HS256})
	if err != nil {
		log.Fatalf("Invalid JWT: %v", err)
	}

	// Declare a variable to store claims.
	claims := jwt.Claims{}

	// Validate the JWT signature and extract claims.
	if err := token.Claims(key, &claims); err != nil {
		log.Fatalf("JWT claims validation failed: %v", err)
	}

	// Additional validations can be performed here.
	fmt.Printf("JWT Claims: %+v\n", claims)
}
```

## Best Practices

- Use strong and unique secret keys for signing tokens (at least 256 bits for HS256), and update periodically to ensure security.
- Store tokens securely, preferably in secure cookies with flags like `HttpOnly` and `Secure`.
- Set appropriate expiry times for your JWTs. Shorter validity periods reduce the impact of a leaked token.
- Always validate the token claims, including audience and issuer, to ensure they conform to expected values.
- Fail fast during initialization if required environment variables are missing rather than using default values.

## Common Pitfalls

- Failing to validate tokens properly can open up security vulnerabilities.
- Not setting a short expiry for each token can increase the risk window in case of a token leak.
- Overlooking the use of HTTPS when transmitting JWTs could lead to token exposure.
- Using weak or predictable secret keys that can be brute-forced.

## Performance Tips

- Consider caching validated tokens to avoid unnecessary recomputation of the same validation logic.
- Large payload data in tokens can increase the size of the token considerably, impacting performance, so keep tokens small.
- Use goroutines for parallel token processing in high-throughput applications to improve performance without blocking operations.