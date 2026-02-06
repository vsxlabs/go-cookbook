---
title: 'Character Encoding in Go'
description: 'Learn how to handle different character encodings in Go using standard and third-party packages'
date: '2025-03-24'
category: 'Localization'
---

Character encoding is a crucial aspect of localization that deals with representing text in computers. In Go, handling character encodings efficiently is key when working with diverse data sources and international text.

## Converting Between UTF-8 and Other Encodings

The Go standard library provides the `golang.org/x/text/encoding` package to manage character encodings beyond UTF-8, which is Go's default.

### Example: Convert from ISO-8859-1 to UTF-8

```go
package main

import (
	"bytes"
	"fmt"
	"io"
	"log"

	"golang.org/x/text/encoding/charmap"
	"golang.org/x/text/transform"
)

func main() {
	// Sample string in ISO-8859-1.
	encodedStr := []byte{0xE7, 0xE9}

	// Create a transformer to convert ISO-8859-1 to UTF-8.
	reader := transform.NewReader(bytes.NewReader(encodedStr), charmap.ISO8859_1.NewDecoder())

	// Read the transformed output.
	decodedBytes, err := io.ReadAll(reader)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(string(decodedBytes)) // Output: cé
}
```

## Using `encoding/json` with Character Encodings

The `encoding/json` package assumes JSON input is UTF-8 encoded, so when dealing with non-UTF-8 data, conversion is necessary before parsing.

### Example: Reading JSON with Different Encoding

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"

	"golang.org/x/text/encoding/charmap"
	"golang.org/x/text/transform"
)

func main() {
	// ISO-8859-1 encoded JSON.
	data := []byte{0x7B, 0x22, 0x6E, 0x61, 0x6D, 0x65, 0x22, 0x3A, 0x22, 0xE9, 0x22, 0x7D} // {"name":"é"}

	// Convert to UTF-8.
	reader := transform.NewReader(bytes.NewReader(data), charmap.ISO8859_1.NewDecoder())
	utf8Data, err := io.ReadAll(reader)
	if err != nil {
		log.Fatal(err)
	}

	var obj map[string]any
	if err := json.Unmarshal(utf8Data, &obj); err != nil {
		log.Fatal(err)
	}

	fmt.Println(obj["name"]) // Output: é
}
```

## Best Practices

- Use the `transform` package for encoding conversions; avoid manually handling byte manipulations.
- Test your application with diverse character encodings to ensure robustness.
- Prefer using UTF-8 as the internal format within your application for consistency and ease of debugging.

## Common Pitfalls

- Assuming all text inputs are in UTF-8; this assumption can lead to unexpected failures and data corruption.
- Forgetting to handle error scenarios when converting text, which can cause runtime crashes.
- Confusing character sets (e.g., ISO) with encodings (e.g., UTF-8).

## Performance Tips

- Use batch processing for large files to minimize reading and conversion overhead.
- Consider lazy loading and conversion on demand to reduce memory footprint.
- Benchmark and profile conversion-heavy parts of your code to identify bottlenecks and optimize accordingly.
