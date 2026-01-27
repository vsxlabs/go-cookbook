---
title: 'HTTPS and TLS Configuration'
description: 'Learn how to securely configure HTTPS and TLS in Go applications with proper certificate handling and security best practices.'
date: '2026-01-26'
category: 'Security'
---

Proper TLS configuration is critical for securing network communications. This guide demonstrates production-ready HTTPS and TLS configurations in Go, with emphasis on security best practices.

## Secure HTTPS Server

A basic HTTPS server with modern TLS settings:

```go
package main

import (
	"crypto/tls"
	"log"
	"net/http"
	"time"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello, secure world!\n"))
	})

	tlsConfig := &tls.Config{
		MinVersion:               tls.VersionTLS13,
		PreferServerCipherSuites: true,
		CurvePreferences: []tls.CurveID{
			tls.X25519,
			tls.CurveP256,
		},
	}

	server := &http.Server{
		Addr:         ":8443",
		Handler:      mux,
		TLSConfig:    tlsConfig,
		ReadTimeout:  15 * time.Second,
		WriteTimeout: 15 * time.Second,
		IdleTimeout:  60 * time.Second,
	}

	log.Println("Starting HTTPS server on :8443")
	if err := server.ListenAndServeTLS("server.crt", "server.key"); err != nil {
		log.Fatal(err)
	}
}
```

**Key security features:**
- Enforces TLS 1.3 minimum version
- Configures strong cipher suites and curve preferences
- Sets appropriate timeouts to prevent resource exhaustion

## HTTPS Client with Proper Certificate Verification

Never use `InsecureSkipVerify=true` in production code. This disables all TLS security and allows Man-in-the-Middle attacks.

Instead, properly configure certificate verification:

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"time"
)

func main() {
	// Create a custom TLS configuration with proper certificate verification.
	tlsConfig := &tls.Config{
		MinVersion: tls.VersionTLS12,
		// NEVER set InsecureSkipVerify to true in production!		
	}

	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: tlsConfig,
			IdleConnTimeout: 90 * time.Second,
		},
		Timeout: 30 * time.Second,
	}

	resp, err := client.Get("https://example.com")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Status: %s\n", resp.Status)
	fmt.Printf("Body length: %d bytes\n", len(body))
}
```

## Handling Self-Signed Certificates Securely

When working with self-signed certificates (e.g., in development or private networks), load the CA certificate into the trusted root pool:

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"time"
)

func main() {
	// Load the self-signed CA certificate.
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		log.Fatalf("Failed to read CA certificate: %v", err)
	}

	// Create a certificate pool and add the CA cert.
	caCertPool := x509.NewCertPool()
	if !caCertPool.AppendCertsFromPEM(caCert) {
		log.Fatal("Failed to append CA certificate to pool")
	}

	// Configure TLS with the custom CA pool.
	tlsConfig := &tls.Config{
		RootCAs:    caCertPool,
		MinVersion: tls.VersionTLS12,
	}

	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: tlsConfig,
		},
		Timeout: 30 * time.Second,
	}

	resp, err := client.Get("https://myserver.local")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(string(body))
}
```

**This approach:**
- Maintains full TLS security
- Validates the certificate chain against your specific CA
- Protects against MitM attacks
- Is safe for production when using private CAs

## Mutual TLS (mTLS) - Client Certificate Authentication

For scenarios requiring client authentication:

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"time"
)

func main() {
	// Load client certificate and private key.
	cert, err := tls.LoadX509KeyPair("client.crt", "client.key")
	if err != nil {
		log.Fatalf("Failed to load client certificate: %v", err)
	}

	// Load CA certificate for server verification.
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		log.Fatalf("Failed to read CA certificate: %v", err)
	}

	caCertPool := x509.NewCertPool()
	if !caCertPool.AppendCertsFromPEM(caCert) {
		log.Fatal("Failed to append CA certificate")
	}

	// Configure TLS with both client cert and CA pool.
	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		RootCAs:      caCertPool,
		MinVersion:   tls.VersionTLS12,
	}

	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: tlsConfig,
		},
		Timeout: 30 * time.Second,
	}

	resp, err := client.Get("https://secure-api.example.com")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(string(body))
}
```

## Server with Client Certificate Verification

Require and verify client certificates on the server side:

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"log"
	"net/http"
	"os"
	"time"
)

func main() {
	// Load CA certificate to verify client certificates.
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		log.Fatalf("Failed to read CA certificate: %v", err)
	}

	caCertPool := x509.NewCertPool()
	if !caCertPool.AppendCertsFromPEM(caCert) {
		log.Fatal("Failed to append CA certificate")
	}

	tlsConfig := &tls.Config{
		ClientAuth:               tls.RequireAndVerifyClientCert,
		ClientCAs:                caCertPool,
		MinVersion:               tls.VersionTLS12,
		PreferServerCipherSuites: true,
		CurvePreferences: []tls.CurveID{
			tls.X25519,
			tls.CurveP256,
		},
	}

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Access client certificate information.
		if r.TLS != nil && len(r.TLS.PeerCertificates) > 0 {
			clientCert := r.TLS.PeerCertificates[0]
			w.Write([]byte("Hello, " + clientCert.Subject.CommonName + "!\n"))
		} else {
			w.Write([]byte("Hello, authenticated client!\n"))
		}
	})

	server := &http.Server{
		Addr:         ":8443",
		Handler:      mux,
		TLSConfig:    tlsConfig,
		ReadTimeout:  15 * time.Second,
		WriteTimeout: 15 * time.Second,
		IdleTimeout:  60 * time.Second,
	}

	log.Println("Starting mTLS server on :8443")
	if err := server.ListenAndServeTLS("server.crt", "server.key"); err != nil {
		log.Fatal(err)
	}
}
```

## Advanced TLS Configuration

Fine-tune TLS settings for specific security requirements:

```go
package main

import (
	"crypto/tls"
	"log"
	"net/http"
	"time"
)

func main() {
	tlsConfig := &tls.Config{
		// Enforce TLS 1.3 only (most secure).
		MinVersion: tls.VersionTLS13,
		MaxVersion: tls.VersionTLS13,

		// Prefer server cipher suite order.
		PreferServerCipherSuites: true,

		// Specify allowed cipher suites (TLS 1.2 and below).
		// Note: TLS 1.3 cipher suites are not configurable.
		CipherSuites: []uint16{
			tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
			tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
			tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
			tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
		},

		// Prefer modern elliptic curves.
		CurvePreferences: []tls.CurveID{
			tls.X25519,
			tls.CurveP256,
			tls.CurveP384,
		},

		// Enable session tickets for resumption.
		SessionTicketsDisabled: false,

		// Renegotiation settings.
		Renegotiation: tls.RenegotiateNever,
	}

	server := &http.Server{
		Addr:              ":8443",
		Handler:           http.DefaultServeMux,
		TLSConfig:         tlsConfig,
		ReadTimeout:       15 * time.Second,
		ReadHeaderTimeout: 5 * time.Second,
		WriteTimeout:      15 * time.Second,
		IdleTimeout:       60 * time.Second,
		MaxHeaderBytes:    1 << 20, // 1 MB.
	}

	log.Fatal(server.ListenAndServeTLS("server.crt", "server.key"))
}
```

## Generating Self-Signed Certificates for Testing

For development and testing purposes, generate certificates using OpenSSL:

```bash
# Generate CA private key.
openssl genrsa -out ca.key 4096

# Generate CA certificate.
openssl req -new -x509 -days 365 -key ca.key -out ca.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=My CA"

# Generate server private key.
openssl genrsa -out server.key 4096

# Generate server certificate signing request
openssl req -new -key server.key -out server.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=localhost"

# Sign server certificate with CA.
openssl x509 -req -days 365 -in server.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out server.crt -sha256

# Generate client certificate (for mTLS).
openssl genrsa -out client.key 4096
openssl req -new -key client.key -out client.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=client"
openssl x509 -req -days 365 -in client.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out client.crt -sha256
```

## Testing TLS Configuration

Test your server's TLS configuration:

```bash
# Test with openssl.
openssl s_client -connect localhost:8443 -CAfile ca.crt

# Test with curl.
curl -v --cacert ca.crt https://localhost:8443

# Test with curl and client certificate (mTLS).
curl -v --cacert ca.crt --cert client.crt --key client.key https://localhost:8443

# Check TLS version and cipher.
nmap --script ssl-enum-ciphers -p 8443 localhost
```

## Best Practices

- Always verify certificates in production.
- Use TLS 1.2 or higher (prefer TLS 1.3).
- Load custom CAs properly when working with self-signed certificates.
- Set appropriate timeouts on servers and clients
- Rotate certificates before expiration
- Use strong cipher suites and curves
- Enable HSTS headers for web applications
- Monitor certificate expiration with automated alerts
- Never use `InsecureSkipVerify: true` in production
- Don't commit certificates or keys to version control
- Don't use TLS 1.0 or 1.1 (deprecated and insecure)
- Don't ignore certificate validation errors
- Don't use weak cipher suites or export-grade cryptography
- Don't reuse certificates across different services unnecessarily

## Common Pitfalls

- **Disabling Certificate Verification**: Never set `InsecureSkipVerify: true` in production as it allows Man-in-the-Middle attacks
- **Not Checking Certificate Loading Errors**: Always verify that `AppendCertsFromPEM()` returns true when loading CA certificates
- **Missing Timeouts**: Set appropriate timeouts on HTTP clients and servers to prevent resource exhaustion

## Performance Considerations

- **Session Resumption**: Enable TLS session tickets for faster reconnections
- **Connection Pooling**: Reuse TLS connections with `http.Client`
- **Hardware Acceleration**: Use CPU with AES-NI for cipher operations
- **TLS 1.3**: Faster handshake with 1-RTT (or 0-RTT for resumption)
- **Keep-Alive**: Enable HTTP keep-alive to avoid repeated handshakes
