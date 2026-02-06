---
title: 'Cross-site Scripting (XSS) Prevention in Go'
description: 'Learn how to prevent Cross-site Scripting (XSS) attacks in Go applications using standard libraries.'
date: '2025-03-24'
category: 'Security'
---

Cross-site Scripting (XSS) can lead to serious security vulnerabilities in web applications. This guide demonstrates how to prevent XSS attacks using standard Go libraries.

## Basic XSS Prevention

When generating HTML using Go's `html/template` package, it automatically escapes data to prevent XSS:

```go
package main

import (
	"html/template"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl := template.Must(template.New("example").Parse(`<h1>Hello, {{.}}</h1>`))
		
		data := r.URL.Query().Get("name")
		tmpl.Execute(w, data)
	})

	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}
```
Using `html/template` ensures that any user input is safely escaped when rendering in HTML.

## Custom HTML Escaping

If you need to escape HTML strings at some point, use `html` package:
```go
package main

import (
	"html"
	"fmt"
)

func main() {
	rawInput := `<script>alert('XSS')</script>`
	escapedInput := html.EscapeString(rawInput)
	fmt.Println("Escaped:", escapedInput)
}
```
The `html.EscapeString()` function escapes special HTML characters, mitigating potential XSS vulnerabilities.

## JSON Binding and XSS

When handling JSON, XSS prevention involves validating and sanitizing data:
```go
package main

import (
	"encoding/json"
	"fmt"
	"html"
)

type UserInput struct {
	Name string `json:"name"`
}

func main() {
	jsonData := `{"name": "<script>alert('XSS')</script>"}`
	var input UserInput
	json.Unmarshal([]byte(jsonData), &input)
	name := html.EscapeString(input.Name) // Escape output
	fmt.Println("Safe Name:", name)
}
```

## Best Practices

- Use `html/template` to render HTML templates in Go; it automatically escapes data.
- Always validate and escape user-supplied input before rendering or processing it, especially when converting user data for HTML output.
- Regularly update libraries since security patches are continuously made.
- Use Content Security Policy (CSP) headers to mitigate the risk of XSS.

## Common Pitfalls

- Directly embedding untrusted data into HTML without escaping.
- Assuming that JSON input will not contribute to XSS — always validate and escape after parsing.
- Overlooking updates to Go libraries and packages that may contain security fixes.

## Performance Tips

- Utilize built-in packages like `html/template` for rendering to leverage parser optimizations and security features.
- Escaping strings is crucial but can affect performance; make sure it is performed at the right stage (output point) in your templates or JSON generation.
- Review and refactor code regularly to ensure that new inputs or changes have not introduced vulnerabilities.

By following these guidelines, Go developers can significantly reduce the risk of XSS attacks while maintaining efficient web application performance.