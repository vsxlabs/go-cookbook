---
title: 'Fuzzing with Go'
description: 'Explore how to perform fuzz testing in Go using built-in fuzzing support'
date: '2025-03-24'
category: 'Testing'
---

Go provides built-in support for fuzz testing, an effective technique used to identify unknown vulnerabilities by inputting random data into your functions. This snippet will guide you through the basics of fuzz testing in Go.

## Getting Started with Go Fuzzing

Go's testing package introduced fuzz testing capabilities in Go 1.18. To begin fuzz testing, create a function prefixed with `Fuzz` in a `_test.go` file.

### Simple Fuzz Test Example

Below is a basic example that tests an integer parsing function using fuzzing:

```go
package main

import (
	"strconv"
	"testing"
)

// Function to be tested.
func ParseInt(s string) (int64, error) {
	return strconv.ParseInt(s, 10, 64)
}

// Fuzz test function.
func FuzzParseInt(f *testing.F) {
	// Providing seed inputs.
	f.Add("123")
	f.Add("-456")
	f.Add("invalid")

	f.Fuzz(func(t *testing.T, orig string) {
		result, err := ParseInt(orig)
		// Check properties that should always hold:
		// If no error, result should be non-zero when input is non-empty and non-zero.
		// If error, that's acceptable for non-numeric input.
		if err == nil {
			// Verify the result makes sense
			t.Logf("Parsed '%s' as %d", orig, result)
		}
	})
}
```

## Running Fuzz Tests

To run fuzz tests, use the `go test` command with the `-fuzz` flag specifying the function pattern to fuzz:

```shell
go test -fuzz=Fuzz
```

Fuzzing will generate various random inputs, checking your code against different edge cases.

## Advanced Fuzzing Features

### Example with Map Input

Fuzz testing can involve complex data types such as maps or structs:

```go
package main

import (
	"strings"
	"testing"
)

// Sample function to be tested.
func RemoveSpaces(s string) string {
	return strings.ReplaceAll(s, " ", "")
}

// Fuzz test for RemoveSpaces.
func FuzzRemoveSpaces(f *testing.F) {
	f.Add("hello world")
	f.Add(" test ")

	f.Fuzz(func(t *testing.T, data string) {
		trimmed := RemoveSpaces(data)
		if strings.Contains(trimmed, " ") {
			t.Errorf("String still contains spaces: %s", trimmed)
		}
	})
}
```

## Best Practices

- Use seed inputs similar to expected and unexpected formats to guide the fuzzing process.
- Ensure your functions handle all possible inputs, including edge cases like empty strings or large numbers.
- Monitor the fuzzing process and adjust based on coverage or observed failures.

## Common Pitfalls

- Overlooking the need to control or specify execution time for fuzz tests, which may lead to long-running tests.
- Failing to provide meaningful seed data could limit the effectiveness of fuzzing.
- Assuming the absence of crashes indicates no errors; thorough log analysis is also needed.

## Performance Tips

- Use fuzzing judiciously when performance might be critical, as fuzzing involves significant randomness and many test iterations.
- Limit the resource allocation and test duration when dealing with large or complex test cases to avoid overwhelming the system.
- Make use of fuzzing deduplication features to filter out redundant inputs that don't increase code coverage.