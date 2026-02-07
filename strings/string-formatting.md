---
title: 'String Formatting in Go'
description: 'Learn how to format strings in Go using built-in functions and best practices.'
date: '2025-03-24'
category: 'Tools'
---

String formatting is an essential task in programming, enabling developers to create readable and dynamic text output. Go provides built-in support for string formatting through the `fmt` package, allowing flexible and easy manipulation of string data.

## Basic String Formatting

Go's `fmt` package offers various methods for formatting strings. Here's a basic example using `Printf` and `Sprintf`:

```go
package main

import (
	"fmt"
)

func main() {
	location := "San Francisco"
	temperature := 22.5

	// Print weather information to standard output.
	fmt.Printf("Location: %s, Temperature: %.1f°C\n", location, temperature)

	// Format weather report as a string for later use.
	report := fmt.Sprintf("Current temperature in %s: %.1f°C", location, temperature)
	fmt.Println(report)
}
```

## Formatting with Different Data Types

The `fmt` package can format a wide variety of data types:

```go
package main

import (
	"fmt"
)

func main() {
	stockPrice := 157.85
	shares := 1000
	isMarketOpen := true

	// Display financial information with appropriate formatting.
	fmt.Printf("Stock Price: $%.2f\n", stockPrice)
	fmt.Printf("Number of Shares: %d\n", shares)
	fmt.Printf("Market Status: %t\n", isMarketOpen)
}
```

## Customizing Number Formats

You can customize numeric formats with width and precision settings:

```go
package main

import (
	"fmt"
)

func main() {
	columnWidth, decimalPlaces := 12, 2
	revenue := 1234567.89

	// Display financial data with various formatting options.
	fmt.Printf("Basic Format: $%f\n", revenue)
	fmt.Printf("Fixed Width: $%*f\n", columnWidth, revenue)
	fmt.Printf("Custom Decimals: $%.2f\n", revenue)
	fmt.Printf("Aligned with Decimals: $%*.*f\n", columnWidth, decimalPlaces, revenue)
}
```

## String Formatting Using `fmt.Printf` and `Sprintf`

You can perform string formatting and interpolation using standard fmt package functions:

```go
package main

import (
	"fmt"
)

func main() {
	city := "Tokyo"
	conditions := "Partly Cloudy"
	humidity := 65

	// Create weather report using string interpolation.
	fmt.Printf("Weather in %v: %v with %v%% humidity.\n", city, conditions, humidity)
	forecast := fmt.Sprintf("Forecast for %v: %v, Humidity: %v%%.", city, conditions, humidity)
	fmt.Println(forecast)
}
```

## Best Practices

- Favor `Sprintf` over `Printf` for constructing strings: Use `Sprintf` when you need to build a string instead of directly printing it.
- Employ `%v` for generic formatting: It provides a default format for most types and is useful when the exact format is not critical.
- Use width and precision only when needed: Over-formatting can make code less readable and may not be necessary unless you're aligning output.

## Common Pitfalls

- Forgetting the newline: `Printf` doesn't automatically append a newline, so remember to include `\n` if needed.
- Misusing format verbs: Always verify the format verb matches the intended data type to avoid runtime errors.
- Invalid width/precision: Setting invalid format specifiers can cause unwanted results or errors.

## Performance Tips

- Use `%s` for string and `%d` for integers: These are less expensive than `%v` as they require fewer type checks.
- Avoid unnecessary precision specification on integers and simple strings.
- Consider pre-allocated buffers with `strings.Builder` for complex string building to minimize allocations.
- Regularly profile your code when working with large or complex string formatting to identify bottlenecks.

By understanding and wisely applying these string formatting techniques, you can write cleaner and more efficient Go code.