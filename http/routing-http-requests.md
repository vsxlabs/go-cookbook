---
title: 'Routing HTTP Requests'
description: 'Learn how to route HTTP requests using both the enhanced standard library and third-party libraries in Go.'
date: '2025-03-24'
category: 'HTTP'
---

Routing HTTP requests efficiently is crucial when building web applications. Go's standard `net/http` package now includes enhanced routing capabilities with method-specific handlers and path wildcards, while third-party libraries like Gorilla Mux provide additional advanced features.

## Standard Library Routing

The standard `net/http` package now supports method-specific handlers and path wildcards for more powerful routing:

```go
package main

import (
	"fmt"
	"log"
	"net/http"	
)

func main() {
	mux := http.NewServeMux()
	
	mux.HandleFunc("POST /users", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Creating a new user")
	})


	// Path wildcards with method specification.
	mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		userID := r.PathValue("id")
		fmt.Fprintf(w, "User ID: %s\n", userID)
	})
	
	
	// Wildcard segments can capture multiple path elements.
	mux.HandleFunc("GET /files/{path...}", func(w http.ResponseWriter, r *http.Request) {
		filePath := r.PathValue("path")
		fmt.Fprintf(w, "File path: %s\n", filePath)
	})
	
	if err := http.ListenAndServe(":8080", mux); err != nil {
		log.Fatal(err)
	}
}
```

## Advanced Standard Library Patterns

Use more sophisticated routing patterns with the enhanced standard library:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"strconv"
)

type User struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

func main() {
	mux := http.NewServeMux()

	// RESTful API routes with method-specific handlers.
	mux.HandleFunc("GET /api/users", listUsers)
	mux.HandleFunc("POST /api/users", createUser)
	mux.HandleFunc("GET /api/users/{id}", getUser)
	mux.HandleFunc("PUT /api/users/{id}", updateUser)
	mux.HandleFunc("DELETE /api/users/{id}", deleteUser)

	// Nested resource routing.
	mux.HandleFunc("GET /api/users/{userID}/posts/{postID}", getUserPost)

	// File serving with wildcard.
	mux.Handle("GET /static/", http.StripPrefix("/static/", http.FileServer(http.Dir("./static/"))))

	if err := http.ListenAndServe(":8080", mux); err != nil {
		log.Fatal(err)
	}
}

func listUsers(w http.ResponseWriter, r *http.Request) {
	users := []User{{1, "Alice"}, {2, "Bob"}}
	json.NewEncoder(w).Encode(users)
}

func createUser(w http.ResponseWriter, r *http.Request) {
	var user User
	json.NewDecoder(r.Body).Decode(&user)
	user.ID = 3 // Simulate ID assignment
	json.NewEncoder(w).Encode(user)
}

func getUser(w http.ResponseWriter, r *http.Request) {
	idStr := r.PathValue("id")
	id, err := strconv.Atoi(idStr)
	if err != nil {
		http.Error(w, "Invalid user ID", http.StatusBadRequest)
		return
	}
	user := User{ID: id, Name: "Example User"}
	json.NewEncoder(w).Encode(user)
}

func updateUser(w http.ResponseWriter, r *http.Request) {
	idStr := r.PathValue("id")
	fmt.Fprintf(w, "Updating user %s\n", idStr)
}

func deleteUser(w http.ResponseWriter, r *http.Request) {
	idStr := r.PathValue("id")
	fmt.Fprintf(w, "Deleting user %s\n", idStr)
}

func getUserPost(w http.ResponseWriter, r *http.Request) {
	userID := r.PathValue("userID")
	postID := r.PathValue("postID")
	fmt.Fprintf(w, "User %s, Post %s\n", userID, postID)
}
```

## Basic Routing with Gorilla Mux

For more advanced routing features, Gorilla Mux remains a popular choice:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"github.com/gorilla/mux"
)

func main() {
	r := mux.NewRouter()

	// Define a simple route.
	r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Welcome to the home page!")
	})

	// Start the server.
	if err := http.ListenAndServe(":8080", r); err != nil {
		log.Fatal(err)
	}
}
```

## Handling URL Parameters

Gorilla Mux makes it easy to capture URL parameters and use them in handler functions.

```go
package main

import (
	"fmt"
	"net/http"
	"github.com/gorilla/mux"
)

func main() {
	r := mux.NewRouter()

	// Define a route with a URL parameter.
	r.HandleFunc("/user/{name}", func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		name := vars["name"]
		fmt.Fprintf(w, "Hello, %s!\n", name)
	})

	// Start the server.
	if err := http.ListenAndServe(":8080", r); err != nil {
		fmt.Println("Server failed:", err)
	}
}
```

## Using Middleware with Gorilla Mux

Middleware functions can be used to execute common logic for multiple routes.

```go
package main

import (
	"fmt"
	"net/http"
	"github.com/gorilla/mux"
)

// Logger middleware.
func logger(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("Request URL:", r.URL)
		next.ServeHTTP(w, r)
	})
}

func main() {
	r := mux.NewRouter()

	// Apply middleware to all routes.
	r.Use(logger)

	r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Welcome to the home page with logging!")
	})

	// Start the server.
	if err := http.ListenAndServe(":8080", r); err != nil {
		fmt.Println("Server failed:", err)
	}
}
```

## Subrouters for Modular Design

Subrouters can help manage routes by grouping them logically, which is useful for complex applications.

```go
package main

import (
	"fmt"
	"net/http"
	"github.com/gorilla/mux"
)

func main() {
	r := mux.NewRouter()

	// Create a subrouter.
	api := r.PathPrefix("/api").Subrouter()
	api.HandleFunc("/status", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "API is running")
	})

	// Start the server.
	if err := http.ListenAndServe(":8080", r); err != nil {
		fmt.Println("Server failed:", err)
	}
}
```

## Best Practices

- Prefer the enhanced standard library routing for most use cases; it's now powerful enough for many applications.
- Use method-specific handlers (`GET /path`, `POST /path`) for cleaner, more explicit routing.
- Leverage path wildcards (`{id}`) and catch-all patterns (`{path...}`) for flexible routing.
- Employ middleware for logging, authentication, and other cross-cutting concerns.
- Use URL parameters wisely to keep endpoints clean and intuitive.
- Organize routes using subrouters to separate different parts of your API.

## Common Pitfalls

- Forgetting to start the server with `http.ListenAndServe`.
- Not closing response bodies in middleware or handlers if necessary.
- Failing to handle errors when retrieving URL params, leading to runtime panics.
- Misconfiguring paths in complex routes, which can lead to unexpected behavior.

## Performance Tips

- Define routes specifically to avoid unnecessary regular expression parsing.
- Minimize middleware layers in performance-critical paths to reduce request processing time.
- Prioritize routes from most specific to least specific to optimize matching speed.
- Reuse handlers where possible to avoid performance costs associated with repeated object creation.