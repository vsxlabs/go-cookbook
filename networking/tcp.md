---
title: 'Working with TCP in Go'
description: 'Learn how to implement TCP clients and servers in Go using the net package'
date: '2025-03-24'
category: 'Networking'
---

TCP (Transmission Control Protocol) is a foundational protocol for reliable communication between network devices. In Go, the `net` package provides robust support for creating TCP clients and servers.

## Implementing a TCP Server

Below is a simple TCP server that listens for incoming connections and echoes received messages back to the client:

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"log"
)

func main() {
	ln, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatal(err)
	}
	defer ln.Close()

	fmt.Println("Server listening on :8080")
	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("Error accepting connection:", err)
			continue
		}

		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()

	reader := bufio.NewReader(conn)
	for {
		message, err := reader.ReadString('\n')
		if err != nil {
			fmt.Println("Error reading message:", err)
			break
		}
		fmt.Print("Received message: ", message)

		_, err = conn.Write([]byte(message))
		if err != nil {
			fmt.Println("Error sending response:", err)
			break
		}
	}
}
```

## Implementing a TCP Client

Here is a basic TCP client that connects to the above server and sends a message:

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
	"log"
)

func main() {
	// Use net.JoinHostPort for IPv6 safety.
	address := net.JoinHostPort("localhost", "8080")
	conn, err := net.Dial("tcp", address)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	fmt.Println("Connected to server, type a message:")
	reader := bufio.NewReader(os.Stdin)

	for {
		message, _ := reader.ReadString('\n')
		_, err := conn.Write([]byte(message))
		if err != nil {
			fmt.Println("Error sending message:", err)
			break
		}

		response, err := bufio.NewReader(conn).ReadString('\n')
		if err != nil {
			fmt.Println("Error receiving response:", err)
			break
		}
		fmt.Print("Response from server: ", response)
	}
}
```

## Best Practices

- Use `defer` to ensure that network connections are closed appropriately.
- Handle connections concurrently using goroutines in the server for scalability.
- Implement proper error handling to handle network failures gracefully.
- Use timeouts (`SetDeadline`) for connections to avoid hanging on I/O operations.
- Always use `net.JoinHostPort` when constructing addresses to ensure IPv6 compatibility.

## Common Pitfalls

- Not closing connections, leading to resource leaks.
- Ignoring the potential for partial reads/writes; always check how many bytes were read/written.
- Failing to handle concurrent access properly; race conditions can occur if shared state is not managed correctly.
- Not setting a read/write deadline, which can cause the application to hang indefinitely.

## Performance Tips

- Use buffered I/O, like `bufio.Reader` and `bufio.Writer`, to improve performance by reducing the number of system calls.
- If high throughput is required, consider limiting the number of active goroutines to avoid resource exhaustion.
- Optimize network usage by considering message sizes and using compression if necessary.
- Profile your application network usage and balance connections across multiple CPUs for better performance.
