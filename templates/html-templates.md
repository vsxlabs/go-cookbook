---
title: 'HTML Templates'
description: 'Discover how to use HTML templates in Go for web development using the html/template package'
date: '2025-03-24'
category: 'Templates'
---

Go provides robust support for HTML templating through the `html/template` package. This guide demonstrates how to use HTML templates effectively to create dynamic web content in Go.

## Basic HTML Template Rendering

Here's a simple example that shows how to load and render an HTML template:

```go
package main

import (
	"html/template"
	"log"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	tmpl, err := template.ParseFiles("template.html")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	data := struct {
		Title string
	}{
		Title: "Hello, Go Templates!",
	}

	err = tmpl.Execute(w, data)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func main() {
	http.HandleFunc("/", handler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Using Template Functions

You can enhance your HTML templates with custom functions:

```go
package main

import (
	"html/template"
	"log"
	"net/http"
	"strings"
)

func handler(w http.ResponseWriter, r *http.Request) {
	funcMap := template.FuncMap{
		"ToUpper": strings.ToUpper,
	}

	tmpl, err := template.New("template.html").Funcs(funcMap).ParseFiles("template.html")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	data := struct {
		Name string
	}{
		Name: "gopher",
	}

	err = tmpl.Execute(w, data)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func main() {
	http.HandleFunc("/", handler)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}
```

## Nested Templates

Create modular templates with inheritance to manage complex layouts:

```go
package main

import (
	"html/template"
	"log"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	templates, err := template.ParseFiles("base.html", "content.html")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	data := struct {
		Content string
	}{
		Content: "Nested Templates in Go",
	}

	err = templates.ExecuteTemplate(w, "base", data)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func main() {
	http.HandleFunc("/", handler)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}
```

## Best Practices

- Always escape HTML inputs to avoid injection attacks
- Use template inheritance and nested templates for better code organization
- Leverage `template.FuncMap` to extend template functionalities
- Keep logic minimal within templates; complex logic should reside in Go code

## Common Pitfalls

- Forgetting to handle errors gracefully, which can expose server internals
- Not considering template caching for improved performance
- Mishandling of unescaped user input, leading to security vulnerabilities
- Repeated parsing of templates instead of reusing parsed templates

## Performance Tips

- Cache parsed templates for reuse to avoid repeated loading and parsing
- Use goroutines to handle simultaneous requests efficiently
- Minimize data passed to templates to avoid large allocations
- Use buffered responses, especially when working with large templates
- Profile your template rendering path to identify bottlenecks in complex applications