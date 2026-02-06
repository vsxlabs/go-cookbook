---
title: 'Basic Data Encoding and Decoding'
description: 'Learn basic techniques for encoding and decoding data in Go using built-in and standard libraries.'
date: '2025-03-24'
category: 'Basics'
---

In Go, data encoding and decoding are fundamental operations for converting data between different formats. This operation is critical for data serialization, network communication, and more. Go's standard library provides robust support for basic encoding and decoding using packages like `encoding/base64`, `encoding/json`, and `encoding/hex`. 

## Base64 Encoding and Decoding

Base64 is commonly used to encode binary data as ASCII text, making it suitable for transmission over text-based protocols.

### Base64 Encoding

```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	data := "Text to encode"
	encodedData := base64.StdEncoding.EncodeToString([]byte(data))
	fmt.Println("Encoded:", encodedData)
}
```

### Base64 Decoding

```go
package main

import (
	"encoding/base64"
	"fmt"
	"log"
)

func main() {
	encodedData := "VGV4dCB0byBlbmNvZGU="
	decodedData, err := base64.StdEncoding.DecodeString(encodedData)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Decoded:", string(decodedData))
}
```

## JSON Encoding and Decoding

JSON is a popular data interchange format. Go provides built-in support via the `encoding/json` package.

### JSON Encoding

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	person := map[string]any{
		"name": "Jack",
		"age":  25,
	}
	jsonData, err := json.Marshal(person)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("JSON Encoded:", string(jsonData))
}
```

### JSON Decoding

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

func main() {
	jsonData := `{"name":"Jack","age":25}`
	var person map[string]any
	err := json.Unmarshal([]byte(jsonData), &person)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("JSON Decoded: %+v\n", person)
}
```

## Hex Encoding and Decoding

Hexadecimal encoding is useful for representing binary data in a readable format.

### Hex Encoding

```go
package main

import (
	"encoding/hex"
	"fmt"
)

func main() {
	data := "Text to encode"
	encodedData := hex.EncodeToString([]byte(data))
	fmt.Println("Hex Encoded:", encodedData)
}
```

### Hex Decoding

```go
package main

import (
	"encoding/hex"
	"fmt"
	"log"
)

func main() {
	encodedData := "5465787420746f20656e636f6465"
	decodedData, err := hex.DecodeString(encodedData)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Hex Decoded:", string(decodedData))
}
```

## Common Pitfalls

- Neglecting error handling can lead to unexpected crashes, especially with malformed data.
- Improper data type assumptions during decoding can cause runtime errors.
- Ignoring character sets when decoding Base64 or Hex could lead to incorrect data interpretation.

## Performance Tips

- For large JSON datasets, consider using `json.Decoder` for streaming instead of processing everything in-memory with `json.Marshal` and `json.Unmarshal`.
- Profile and benchmark different encoding strategies if encoding/decoding is a bottleneck in your application.