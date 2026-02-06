---
title: 'AST Manipulation'
description: 'Learn how to manipulate Abstract Syntax Trees (AST) in Go using the go/ast and go/token packages'
date: '2025-03-24'
category: 'Other topics'
---

Go provides powerful capabilities for manipulating source code using Abstract Syntax Trees (AST). This guide covers the basics of traversing and modifying ASTs with Go's `go/ast` and `go/token` packages.

## Basic AST Traversal

Here's a simple example that shows how to parse a Go source file and traverse its AST:

```go
package main

import (
	"fmt"
	"go/ast"
	"go/parser"
	"go/token"
	"log"
)

func main() {
	src := `
	package main

	func main() {
		fmt.Println("Hello, Go AST!")
	}
	`

	// Create a new token.FileSet for tracking position information.
	fset := token.NewFileSet()

	// Parse the source code and generate an AST.
	node, err := parser.ParseFile(fset, "", src, parser.AllErrors)
	if err != nil {
		log.Fatal(err)
	}

	// Traverse the AST.
	ast.Inspect(node, func(n ast.Node) bool {
		switch x := n.(type) {
		case *ast.FuncDecl:
			fmt.Printf("Function declaration: %s\n", x.Name.Name)
		}
		return true
	})
}
```

## Modifying AST Nodes

Modify AST nodes to transform code. Here's an example that changes function names:

```go
package main

import (
	"go/ast"
	"go/parser"
	"go/printer"
	"go/token"
	"log"
	"os"
)

func main() {
	src := `
	package main

	func oldName() {
		fmt.Println("Old function!")
	}
	`

	fset := token.NewFileSet()
	node, err := parser.ParseFile(fset, "", src, parser.AllErrors)
	if err != nil {
		log.Fatal(err)
	}

	// Modify function names.
	ast.Inspect(node, func(n ast.Node) bool {
		if fn, ok := n.(*ast.FuncDecl); ok {
			if fn.Name.Name == "oldName" {
				fn.Name.Name = "newName"
			}
		}
		return true
	})

	// Print the modified code.
	printer.Fprint(os.Stdout, fset, node)
}
```

## AST Node Insertion

Insert new nodes to extend an AST.

```go
package main

import (
	"go/ast"
	"go/parser"
	"go/token"
	"go/printer"
	"os"
	"log"
)

func main() {
	src := `
	package main

	func main() {
		fmt.Println("Start")
	}
	`

	fset := token.NewFileSet()
	node, err := parser.ParseFile(fset, "", src, parser.AllErrors)
	if err != nil {
		log.Fatal(err)
	}

	// Create a new statement to insert.
	newCall := &ast.ExprStmt{
		X: &ast.CallExpr{
			Fun:  ast.NewIdent("fmt.Println"),
			Args: []ast.Expr{ast.NewIdent("\"New statement\"")},
		},
	}

	// Insert the new statement into the function body.
	for _, decl := range node.Decls {
		if fn, ok := decl.(*ast.FuncDecl); ok && fn.Name.Name == "main" {
			fn.Body.List = append(fn.Body.List, newCall)
		}
	}

	// Print the modified code.
	printer.Fprint(os.Stdout, fset, node)
}
```

## Best Practices

- Always use `token.FileSet` to maintain accurate position information.
- Use `ast.Inspect` for simplified AST traversal.
- Update nodes in a depth-first order to ensure dependent nodes are processed correctly.
- Maintain code formatting using `go/printer` to output modified ASTs neatly.

## Common Pitfalls

- Overlooking token position management can lead to incorrect code transformations.
- Forgetting to validate node types before casting can cause runtime panics.
- Assuming tree node structures remain constant across different Go versions.

## Performance Tips

- For large codebases, leverage parallel processing when analyzing multiple files.
- Cache results from `token.FileSet` as it can be reused for multiple file parses.
- Avoid excessive AST traversals by combining related transformations within a single pass.
