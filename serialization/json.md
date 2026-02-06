---
title: 'Working with JSON in Go'
description: 'Learn how to serialize and deserialize JSON in Go using the encoding/json package'
date: '2025-03-24'
category: 'Serialization'
---

JSON is a popular data interchange format due to its simplicity and readability. Go provides built-in support for encoding and decoding JSON using the `encoding/json` package. This guide provides examples for working with JSON in Go, suitable for both beginners and experienced engineers.

## Basic JSON Serialization

Here's a simple example that demonstrates how to encode a Go struct to a JSON string:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	Age   int    `json:"age"`
}

func main() {
	u := User{Name: "Alice", Email: "alice@example.com", Age: 30}

	// Serialize struct to JSON.
	userJSON, err := json.Marshal(u)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(userJSON))
}
```

## JSON Deserialization

Deserializing JSON into a struct is straightforward using the `Unmarshal` function:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	Age   int    `json:"age"`
}

func main() {
	data := `{"name": "Bob", "email": "bob@example.com", "age": 25}`

	var u User
	// Deserialize JSON string to struct.
	err := json.Unmarshal([]byte(data), &u)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Name: %s, Email: %s, Age: %d\n", u.Name, u.Email, u.Age)
}
```

## Handling Dynamic JSON

If the structure of JSON is not known at compile time, a `map` or `interface{}` can be used:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

func main() {
	data := `{"name": "Charlie", "email": "charlie@example.com", "age": 29, "extra": "value"}`

	var result map[string]any
	err := json.Unmarshal([]byte(data), &result)
	if err != nil {
		log.Fatal(err)
	}

	for key, value := range result {
		fmt.Printf("%s: %v\n", key, value)
	}
}
```

## Experimental JSON v2 Package

The Go team has developed an experimental v2 implementation of `encoding/json` available as `encoding/json/v2` in the standard library. This package can be enabled using the `GOEXPERIMENT=jsonv2` build flag and aims to provide better performance, more flexibility, and improved usability while maintaining mostly backwards compatibility with v1.

### Enabling JSON v2

To use the experimental JSON v2 package, you need to enable it with the `GOEXPERIMENT` flag:

```bash
GOEXPERIMENT=jsonv2 go build myapp.go
GOEXPERIMENT=jsonv2 go run myapp.go
```

Key features of JSON v2 include:

- **Enhanced Performance**: Streaming-first design reduces memory allocation and improves speed
- **Configurable Behavior**: Rich set of options to customize serialization and deserialization
- **Better Error Context**: More detailed error messages with precise location information
- **Strict Field Matching**: Case-sensitive field names by default with optional case-insensitive mode
- **Zero Value Control**: Fine-grained control over how zero values are handled during marshaling

JSON v2 offers powerful configuration options for fine-tuned control:

```go
package main

import (
	"fmt"
	"encoding/json/v2"
	"log"
)

type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email"`
	Age      int      `json:"age"`
	Hobbies  []string `json:"hobbies"`
	IsActive bool     `json:"is_active"`
}

func main() {
	// User with some zero values.
	u := User{
		Name:     "Ben",
		Email:    "ben@example.com",
		Age:      0,        // zero value
		Hobbies:  nil,      // nil slice
		IsActive: false,    // zero value bool
	}

	// Configure marshaling behavior.
	data, err := json.Marshal(u, 
		json.FormatNilSliceAsNull(true),     // nil slices become null
		json.OmitZeroStructFields(true),     // skip zero values
		json.Deterministic(true),            // consistent field ordering
	)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Configured output: %s\n", data)

	// Demonstrate case-insensitive parsing.
	mixedCaseJSON := `{"NAME": "Ray", "EMAIL": "ray@test.com", "AGE": 55}`
	var result User
	err = json.Unmarshal([]byte(mixedCaseJSON), &result,
		json.MatchCaseInsensitiveNames(true),
	)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Parsed from mixed case: %+v\n", result)
}
```

### Custom Serialization with v2

JSON v2 introduces enhanced interfaces for custom marshaling and unmarshaling:

```go
package main

import (
	"fmt"
	"time"
	"encoding/json/v2"
	"encoding/json/v2/jsontext"
	"log"
)

type CustomDate time.Time

// UnmarshalerFrom provides direct decoder access for efficiency
func (d *CustomDate) UnmarshalJSONFrom(decoder *jsontext.Decoder) error {
	var dateStr string
	if err := decoder.ReadValue().Unmarshal(&dateStr); err != nil {
		return err
	}
	parsed, err := time.Parse("2006-01-02", dateStr)
	if err != nil {
		return err
	}
	*d = CustomDate(parsed)
	return nil
}

// MarshalerTo writes directly to encoder for better performance
func (d CustomDate) MarshalJSONTo(encoder *jsontext.Encoder) error {
	formatted := time.Time(d).Format("2006-01-02")
	return encoder.WriteValue(jsontext.Value(`"` + formatted + `"`))
}

type User struct {
	Name      string     `json:"name"`
	Email     string     `json:"email"`
	Age       int        `json:"age"`
	BirthDate CustomDate `json:"birth_date"`
}

func main() {
	user := User{
		Name:      "Eve",
		Email:     "eve@example.com",
		Age:       30,
		BirthDate: CustomDate(time.Date(1974, 7, 10, 0, 0, 0, 0, time.UTC)),
	}

	// Serialize with custom date formatting
	data, err := json.Marshal(user)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Serialized: %s\n", data)

	// Parse back with custom date parsing
	var parsed User
	err = json.Unmarshal(data, &parsed)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Parsed user: %+v\n", parsed)
	fmt.Printf("Birth date as time: %v\n", time.Time(parsed.BirthDate))
}
```

### Migration Considerations

Before adopting JSON v2 in your projects, consider these important factors:

1. **Experimental Nature**: API stability is not guaranteed and changes happen frequently
2. **Build Configuration**: Requires `GOEXPERIMENT=jsonv2` environment variable during compilation
3. **Import Changes**: Switch from `encoding/json` to `encoding/json/v2` in your code
4. **Default Behavior Changes**: Field matching is case-sensitive by default, unlike v1
5. **Performance Gains**: Expect faster processing, particularly for large JSON documents
6. **Backward Compatibility**: Most v1 code works with v2, but some edge cases may differ

## Best Practices

- Use `omitempty` in JSON tags to omit zero-value fields from output whenever necessary.
- Consider using custom JSON marshalling/unmarshalling if custom logic is required.

## Common Pitfalls

- Ignoring error returns from `Marshal` and `Unmarshal`.
- Using incorrect struct tags or forgetting to include them.
- Forgetting to use pointer receivers for `Unmarshal` to modify the original struct.
- Not accounting for different JSON field name case sensitivity.

## Performance Tips

- Avoid using `interface{}` when possible as it incurs type assertion overhead.
- Pre-allocate slices when the size is known, especially for large JSON arrays.
- For performance-critical applications, minimize the use of reflection by ensuring structs match JSON schema closely.
- Use streaming JSON decoding with `json.Decoder` for large JSON data to reduce memory overhead.
- Profile and optimize bottlenecks using Go's profiling tools, especially when dealing with high volumes of JSON data.
