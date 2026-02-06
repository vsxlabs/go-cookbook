---
title: 'Using Structure Annotations'
description: 'Learn how to use structure annotations in Go to control JSON marshalling and database interactions'
date: '2025-03-24'
category: 'Basics'
---

Structure annotations in Go are a powerful way to control how data is serialized and deserialized, as well as how it interacts with databases. This guide will introduce you to using structure annotations with a focus on JSON handling and database tags.

## JSON Annotations

In Go, the `encoding/json` package allows you to annotate struct fields to control JSON marshalling/unmarshalling. The `json` tag is used to specify the JSON key name.

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type Person struct {
	Name  string `json:"name"`
	Age   int    `json:"age"`
	Email string `json:"email,omitempty"` // omitempty skips the field if it's empty.
}

func main() {
	data := `{"name":"Alice","age":25}`
	var p Person
	if err := json.Unmarshal([]byte(data), &p); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Person: %+v\n", p)

	jsonData, err := json.Marshal(p)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(jsonData))
}
```

## Best Practices

- Use `omitempty` in JSON annotations to avoid serializing empty fields whenever necessary.
- Clearly name JSON tags to ensure consistent API naming conventions.

## Common Pitfalls

- Forgetting to export struct fields (capitalize the first letter) which results in JSON and database serialization issues.
- Using incorrect or misspelled tag keys like `jason:"name"` instead of `json:"name"`.
- Not handling `null` values properly when unmarshalling JSON into pointers.

## Performance Tips

- Use `omitempty` to reduce the size of JSON data sent over networks.
- Avoid deep marshalling/unmarshalling loops which may consume significant CPU resources. Consider flattening nested structures if possible.
