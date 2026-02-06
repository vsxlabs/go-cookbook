---
title: 'Implementing REST APIs'
description: 'Learn how to build RESTful APIs in Go using the net/http package'
date: '2025-03-24'
category: 'HTTP'
---

Building REST APIs in Go can be efficiently managed using the built-in `net/http` package. This guide demonstrates how to implement a simple RESTful web service.

## Basic REST API Setup

Here is an example of setting up a basic HTTP server with a RESTful endpoint:

```go
package main

import (
	"encoding/json"	
	"log"
	"net/http"	
)

type Item struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

func getItems(w http.ResponseWriter, r *http.Request) {
	items := []Item{{ID: "1", Name: "Item One"}, {ID: "2", Name: "Item Two"}}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(items)
}

func main() {
	http.HandleFunc("/items", getItems)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}
```

## Adding New Endpoints

To expand functionality, let's add endpoints to create a new item and retrieve an item by ID:

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"sync"
)

type Item struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

var (
	items = make(map[string]Item)
	mu    sync.Mutex
)

func getItemByID(w http.ResponseWriter, r *http.Request) {
	id := r.URL.Query().Get("id")
	mu.Lock()
	item, exists := items[id]
	mu.Unlock()

	if !exists {
		http.NotFound(w, r)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(item)
}

func createItem(w http.ResponseWriter, r *http.Request) {
	var item Item
	if err := json.NewDecoder(r.Body).Decode(&item); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	mu.Lock()
	items[item.ID] = item
	mu.Unlock()

	w.WriteHeader(http.StatusCreated)
}

func main() {
	http.HandleFunc("/item", getItemByID)
	http.HandleFunc("/items", createItem)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}
```

## Using a Router Library

For more advanced routing, consider using a third-party library like `gorilla/mux`:

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"sync"

	"github.com/gorilla/mux"
)

type Item struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

var (
	items = make(map[string]Item)
	mu    sync.Mutex
)

func getItemByID(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	id := vars["id"]
	mu.Lock()
	item, exists := items[id]
	mu.Unlock()

	if !exists {
		http.NotFound(w, r)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(item)
}

func createItem(w http.ResponseWriter, r *http.Request) {
	var item Item
	if err := json.NewDecoder(r.Body).Decode(&item); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	mu.Lock()
	items[item.ID] = item
	mu.Unlock()

	w.WriteHeader(http.StatusCreated)
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/items", createItem).Methods("POST")
	r.HandleFunc("/items/{id}", getItemByID).Methods("GET")
	if err := http.ListenAndServe(":8080", r); err != nil {
		log.Fatal(err)
	}
}
```

## Best Practices

- Use `json.Encoder` and `json.Decoder` to manage JSON serialization to/from input and output streams
- Protect shared state with synchronization primitives such as mutexes

## Common Pitfalls

- Forgetting to set `Content-Type` headers when responding with JSON
- Not managing concurrency safely when accessing shared resources
- Not handling HTTP methods correctly, resulting in incorrect API behavior
- Using the default `net/http` multiplexer for complex routing requirements

## Performance Tips

- Use the `http.Server`'s ReadTimeout and WriteTimeout to manage long-running requests
- Serve static assets or caching frequently accessed endpoints for reducing server load
- Utilize middleware for tasks such as logging and authentication to keep handlers clean
- Profile your API using `pprof` or a similar tool to identify bottlenecks and optimize performance