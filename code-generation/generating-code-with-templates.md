---
title: 'Generating Code with Templates'
description: 'Learn how to generate Go code using text templates for dynamic content and code generation tasks.'
date: '2025-03-24'
category: 'Code Generation'
---

The Go programming language provides powerful templating support through the `text/template` and `html/template` packages. These packages can be used for generating code by composing templates that render dynamic content. This is particularly useful for code generators, report creation, and template-based system configuration.

## Basic Code Generation with text/template

Here's a simple example that demonstrates how to generate code using the `text/template` package:

```go
package main

import (
	"log"
	"os"
	"text/template"
)

func main() {
	const tpl = `
package main

import "fmt"

func main() {
	fmt.Println("Hello, {{.Name}}!")
}
`
	data := struct {
		Name string
	}{
		Name: "World",
	}

	tmpl, err := template.New("hello").Parse(tpl)
	if err != nil {
		log.Fatal(err)
	}

	if err = tmpl.Execute(os.Stdout, data); err != nil {
		log.Fatal(err)
	}
}
```

## Advanced Custom Template Functions

You can enhance templates with custom functions to extend functionality:

```go
package main

import (	
	"log"
	"os"
	"strings"
	"text/template"
)

func main() {
	const tpl = `
package main

import "strings"

func main() {
	uppercase := {{. | uppercase}}
	fmt.Println(uppercase)
}
`

	funcMap := template.FuncMap{
		"uppercase": strings.ToUpper,
	}

	data := "hello, world"

	tmpl, err := template.New("uppercase").Funcs(funcMap).Parse(tpl)
	if err != nil {
		log.Fatal(err)
	}

	err = tmpl.Execute(os.Stdout, data)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Generating Structs from Templates

You can also generate Go structs with predefined fields from templates:

```go
package main

import (
	"log"
	"os"
	"text/template"
)

func main() {
	const tpl = `
package models

type {{.StructName}} struct {
	{{range .Fields}}
	{{.Name}} {{.Type}}
	{{end}}
}
`
	type Field struct {
		Name string
		Type string
	}

	data := struct {
		StructName string
		Fields     []Field
	}{
		StructName: "User",
		Fields: []Field{
			{"ID", "int"},
			{"Name", "string"},
			{"Email", "string"},
		},
	}

	tmpl, err := template.New("struct").Parse(tpl)
	if err != nil {
		log.Fatal(err)
	}

	err = tmpl.Execute(os.Stdout, data)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Best Practices

- Leverage `template.FuncMap` to enhance the utility of your templates by adding custom functions.
- Aim to keep templates as simple as possible but powerful by using logical constructs {like if/else} and iteration wisely.
- Store templates in separate files for ease of management and reuse, loading them using `template.ParseFiles`.

## Common Pitfalls

- Avoid ignoring errors from template parsing or execution, as they can contain valuable information about syntax issues.
- Be mindful of template injection vulnerabilities, particularly when using templates with user-provided data in web applications.
- Ensure your templates do not become overly complex; complex templates can be hard to maintain and debug.

## Performance Tips

- Compile your templates once and reuse them if possible, as parsing can be expensive for large templates.
- Use buffers to collect template output rather than writing directly to `os.Stdout`, especially in performance-sensitive contexts.
- Consider caching generated code if you're repeating generation tasks with similar data for better performance.