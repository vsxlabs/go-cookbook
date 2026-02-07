---
title: 'Working with SQLite Databases'
description: 'Learn how to interact with SQLite databases in Go using the github.com/mattn/go-sqlite3 package'
date: '2025-03-24'
category: 'Databases'
---

SQLite is a self-contained, serverless SQL database engine, which is ideal for local storage in applications. Go developers can leverage the `github.com/mattn/go-sqlite3` package to work with SQLite databases efficiently.

## Setting Up SQLite in Go

Here's a simple example that demonstrates how to set up a connection to an SQLite database and perform some basic operations:

```go
package main

import (
	"database/sql"
	"fmt"
	"log"

	_ "github.com/mattn/go-sqlite3"
)

func main() {
	// Open a new connection to the SQLite database.
	db, err := sql.Open("sqlite3", "./test.db")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	// Execute a simple query to create a table.
	_, err = db.Exec("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
	if err != nil {
		log.Fatal(err)
	}

	// Insert a new user.
	_, err = db.Exec("INSERT INTO users (name) VALUES (?)", "Alice")
	if err != nil {
		log.Fatal(err)
	}

	// Read the users back.
	rows, err := db.Query("SELECT id, name FROM users")
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	// Iterate over the result set.
	for rows.Next() {
		var id int
		var name string
		err := rows.Scan(&id, &name)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Printf("User ID: %d, Name: %s\n", id, name)
	}

	// Check for errors from iterating over rows.
	if err = rows.Err(); err != nil {
		log.Fatal(err)
	}
}
```

## Handling Transactions

Transactions in SQLite can be managed using the `Begin`, `Commit`, and `Rollback` methods:

```go
package main

import (
	"database/sql"
	"log"

	_ "github.com/mattn/go-sqlite3"
)

func main() {
	db, err := sql.Open("sqlite3", "./test.db")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	// Start a transaction.
	tx, err := db.Begin()
	if err != nil {
		log.Fatal(err)
	}

	// Perform some operations within the transaction.
	_, err = tx.Exec("INSERT INTO users (name) VALUES (?)", "Bob")
	if err != nil {
		tx.Rollback()
		log.Fatal(err)
	}

	_, err = tx.Exec("INSERT INTO users (name) VALUES (?)", "Charlie")
	if err != nil {
		tx.Rollback()
		log.Fatal(err)
	}

	// Commit the transaction.
	if err := tx.Commit(); err != nil {
		log.Fatal(err)
	}
}
```

## Best Practices

- Always close database connections using `defer` to avoid resource leaks.
- Use prepared statements (`db.Prepare`) to prevent SQL injection and improve performance.
- Handle errors gracefully instead of using `panic` in production code.
- Plan and manage transactions carefully to ensure data consistency.

## Common Pitfalls

- Forgetting to close result sets, which can lead to resource exhaustion.
- Mismanaging transactions, leading to data inconsistencies.
- Not checking for errors on `rows.Next()` which can hide iteration issues.
- Using `panic` for error handling can lead to crashes in production environments.

## Performance Tips

- Use transactions for batch inserts and updates to improve performance.
- Limit the number of rows fetched using SQL `LIMIT` when possible to reduce memory usage.
- Consider using indices on frequently queried columns to speed up lookups.
