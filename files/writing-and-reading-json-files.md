---
title: 'Writing and Reading JSON Files'
description: 'Learn how to efficiently write and read JSON files in Go using the encoding/json package'
date: '2025-03-24'
category: 'Files'
---

Go offers comprehensive support for JSON encoding and decoding through the `encoding/json` package. This snippet demonstrates how to write to and read from JSON files efficiently.

## Writing JSON to a File

Here's a straightforward example showing how to marshal Go data structures into JSON and write it to a file:

```go
package main

import (
	"encoding/json"
	"os"
	"log"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	person := Person{Name: "John Doe", Age: 30}
	
	file, err := os.Create("person.json")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	encoder := json.NewEncoder(file)
	if err := encoder.Encode(person); err != nil {
		log.Fatal(err)
	}
}
```

## Reading JSON from a File

This example shows how to read and unmarshal JSON data from a file into Go data structures:

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
	"log"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	file, err := os.Open("person.json")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	var person Person
	decoder := json.NewDecoder(file)
	if err := decoder.Decode(&person); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Name: %s, Age: %d\n", person.Name, person.Age)
}
```

## Best Practices

- Use `json.NewEncoder` and `json.NewDecoder` with files to avoid loading the entire JSON content into memory.
- Define struct types with `json` tags for seamless JSON marshalling/unmarshalling.
- Always use `defer` to ensure files are closed.
- Handle errors appropriately; avoid using `panic` in production code.

## Common Pitfalls

- Forgetting to close files, which can lead to resource leaks.
- Not using JSON tags, leading to unexpected JSON output.
- Ignoring error handling, leading to undetected issues.
- Assuming JSON data will always match the expected structure.

## Performance Tips

- For large JSON files, stream processing using `json.Decoder` rather than loading all data into memory with `os.ReadFile`.
- Use buffered I/O (`bufio`) for better performance when reading or writing large files.
- Minimize memory footprint by using `interface{}` for unmarshalling only when necessary, and prefer working with specific data types.
