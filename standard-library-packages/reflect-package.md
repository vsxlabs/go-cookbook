---
title: 'Exploring the reflect Package'
description: 'Understand how to use the reflect package in Go for runtime type reflection.'
date: '2025-03-24'
category: 'Standard library packages'
---

The `reflect` package in Go provides the ability to inspect types at runtime. It's a powerful feature that lets you work with types and values dynamically, overcoming the statically-typed nature of Go under certain circumstances.

## Inspecting Types and Values

To use the `reflect` package, you need to understand the concepts of `reflect.Type` and `reflect.Value`.

```go
package main

import (
	"fmt"
	"reflect"
)

func printTypeAndValue(v any) {
	val := reflect.ValueOf(v)
	fmt.Printf("Type: %s, Value: %v\n", val.Type(), val.Interface())
}

func main() {
	x := 42
	y := "reflect package"
	z := 3.14

	printTypeAndValue(x)
	printTypeAndValue(y)
	printTypeAndValue(z)
}
```

## Modifying Values

With `reflect`, you can modify the values of variables at runtime if you have a pointer to them.

```go
package main

import (
	"fmt"
	"reflect"
)

func setValueToZero(ptr any) {
	val := reflect.ValueOf(ptr)
	if val.Kind() == reflect.Ptr && !val.IsNil() {
		elem := val.Elem()
		if elem.CanSet() {
			elem.Set(reflect.Zero(elem.Type()))
		}
	}
}

func main() {
	a := 100
	fmt.Println("Before:", a)
	setValueToZero(&a)
	fmt.Println("After:", a)
}
```

## Working with Structs

Reflection is often used to work with structs, whether reading field values or setting them.

```go
package main

import (
	"fmt"
	"reflect"
)

type Person struct {
	Name string
	Age  int
}

func printStructFields(s any) {
	val := reflect.ValueOf(s)
	typeOfS := val.Type()

	for i := 0; i < val.NumField(); i++ {
		field := val.Field(i)
		fmt.Printf("Field %d: %s = %v\n", i, typeOfS.Field(i).Name, field)
	}
}

func main() {
	p := Person{"John", 30}
	printStructFields(p)
}
```

## Best Practices

- Use the `reflect` package sparingly as it can lead to code that’s hard to read and maintain.
- Always check the kind of a `reflect.Value` before interacting with it to avoid runtime panics.
- Use reflection for tasks that cannot be performed with static typing, such as frameworks or libraries that require deep inspection of types.

## Common Pitfalls

- Modifying immutable objects or non-pointer values using `reflect` will result in a panic.
- Remember that reflective operations are more expensive than normal type operations due to the overhead of dynamic type resolution.
- Avoid using `reflect` for type conversions. Instead, use normal Go type conversion methods when possible.

## Performance Tips

- Cache any reflection-based evaluations if used repeatedly to minimize the performance overhead.
- Consider the cost of reflection against the benefits when choosing between reflective and non-reflective code paths.
- Profiling your program can help identify hotspots where reflection usage might be impacting performance.
