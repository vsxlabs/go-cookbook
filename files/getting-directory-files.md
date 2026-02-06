---
title: 'Getting Directory Files'
description: 'Learn how to retrieve and list files in a directory using Go'
date: '2025-03-24'
category: 'Files'
---

Retrieving the list of files in a directory is a common task that can be easily accomplished in Go using the `os` package. Below are examples demonstrating different approaches to listing files in a directory.

## Basic Directory Listing

Here's a simple way to list files in a directory using `os.ReadDir`, introduced in Go 1.16:

```go
package main

import (
	"fmt"
	"os"
	"log"
)

func main() {
	files, err := os.ReadDir(".") // List files in the current directory
	if err != nil {
		log.Fatal(err)
	}

	for _, file := range files {
		fmt.Println(file.Name())
	}
}
```

## Listing Files with Additional File Info

If you need more information about each file, such as size or modification time, use `os.FileInfo`:

```go
package main

import (
	"fmt"
	"os"
	"log"
)

func main() {
	dirEntries, err := os.ReadDir(".")
	if err != nil {
		log.Fatal(err)
	}

	for _, entry := range dirEntries {
		info, err := entry.Info()
		if err != nil {
			log.Fatal(err)
		}
		fmt.Printf("Name: %s, Size: %d bytes, IsDir: %t\n", info.Name(), info.Size(), info.IsDir())
	}
}
```

## Recursively Listing Files in a Directory

To list files in a directory and its subdirectories, you can use recursion:

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
	"log"
)

func main() {
	root := "." // Starting directory

	err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		fmt.Println(path)
		return nil
	})
	if err != nil {
		log.Fatal(err)
	}
}
```

## Best Practices

- Always handle errors when reading directories or file metadata to avoid unexpected program crashes.
- Use `defer` to handle any resource cleanup, such as closing files if applicable.
- Ensure your program handles symbolic links appropriately if they're part of the directory structure.

## Common Pitfalls

- Trying to read directories without permissions, which will cause an error.
- Ignoring errors returned by functions like `os.ReadDir` or `os.FileInfo`.
- Forgetting to check `info.IsDir()` if you need to filter out directories from your file list.

## Performance Tips

- Use `os.ReadDir` for a more efficient and safer way to list directories.
- For extensive directory structures, consider using buffered output to improve performance.
- Parallelize the processing of files if additional operations are performed on them.
- Minimize the number of allocations by reusing slices or pre-allocating based on an estimated number of files.
