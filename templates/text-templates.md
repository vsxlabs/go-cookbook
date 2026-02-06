---
title: 'Text Templates'
description: 'Learn how to generate and manage text templates in Go using the text/template package'
date: '2025-03-24'
category: 'Templates'
---

Go provides an in-depth text templating system through the `text/template` package, which allows for generating and formatting text with templates. This guide demonstrates basic and advanced text templating techniques.

## Basic Text Templating

Here's an example that demonstrates how to use a simple text template to format a string:

```go
package main

import (
	"os"
	"text/template"
	"log"
)

func main() {
	// Define a template string.
	tmpl := "Hello, {{.}}!"

	// Parse the template string.
	t, err := template.New("hello").Parse(tmpl)
	if err != nil {
		log.Fatal(err)
	}

	// Execute the template with data.
	err = t.Execute(os.Stdout, "World")
	if err != nil {
		log.Fatal(err)
	}
}
```

## Text Templating with Struct Data

You can pass more complex data structures to your templates, such as structs:

```go
package main

import (
	"os"
	"text/template"
	"log"
)

type User struct {
	Name  string
	Email string
}

func main() {
	// Define a template string.
	tmpl := "Name: {{.Name}}, Email: {{.Email}}"

	// Parse the template string.
	t, err := template.New("user").Parse(tmpl)
	if err != nil {
		log.Fatal(err)
	}

	// Create a user struct.
	user := User{Name: "John Doe", Email: "john@example.com"}

	// Execute the template with struct data.
	err = t.Execute(os.Stdout, user)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Using Template Functions

The `text/template` package allows you to define and use functions within templates to enhance formatting capabilities:

```go
package main

import (
	"os"
	"strings"
	"text/template"
	"log"
)

func main() {
	// Define a template string with a function.
	tmpl := "Message: {{upper .}}"

	// Create a template and register functions.
	funcMap := template.FuncMap{
		"upper": strings.ToUpper,
	}
	t, err := template.New("msg").Funcs(funcMap).Parse(tmpl)
	if err != nil {
		log.Fatal(err)
	}

	// Execute the template with data.
	err = t.Execute(os.Stdout, "hello world")
	if err != nil {
		log.Fatal(err)
	}
}
```

## Best Practices

- Use meaningful variable names in templates for readability.
- Always check and handle errors, particularly when parsing templates or processing data.
- Define commonly used template parts as separate templates and use `template.ParseFiles()` or `template.ParseGlob()` for modularization.

## Common Pitfalls

- Forgetting to call `template.Funcs()` before parsing templates when adding custom functions.
- Not handling template execution errors, which can then go unnoticed.
- Hardcoding field names in templates that may change in the struct definition, leading to runtime errors.

## Performance Tips

- Reuse parsed templates whenever possible to reduce parsing overhead during execution.
- Use `template.Must()` for parsing templates if you are sure they won’t encounter errors; this can simplify error handling for static templates loaded at application start.
- Cache templates if they are used repeatedly with different data inputs to gain performance benefits.
