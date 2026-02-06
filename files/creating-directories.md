---
title: 'Creating Directories'
description: 'Learn how to create directories in Go using the os package'
date: '2025-03-24'
category: 'Files'
---

Creating directories in Go can be easily accomplished using the `os` package, which provides functions to create one or multiple directories. This guide demonstrates how you can create directories using Go.

## Basic Directory Creation

Here's a simple example that shows how to create a single directory:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	dirName := "exampleDir"
	err := os.Mkdir(dirName, 0755)
	if err != nil {
		fmt.Println("Error creating directory:", err)
		os.Exit(1)
	}

	fmt.Println("Directory created successfully")
}
```

## Creating Nested Directories

To create a series of directories (nested directories), you can use `os.MkdirAll`:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	nestedDirPath := "exampleDir/nestedDir/anotherDir"
	err := os.MkdirAll(nestedDirPath, 0755)
	if err != nil {
		fmt.Println("Error creating directory path:", err)
		os.Exit(1)
	}

	fmt.Println("Nested directories created successfully")
}
```

## Best Practices

- Always check for errors when creating directories to handle cases where the directory already exists or permissions are insufficient.
- Set the directory permissions appropriately. The permission `0755` is typical for directories, allowing read/write/execute for the owner and read/execute for others.
- Use `os.MkdirAll` when you need to ensure that the entire path is created, not just the last directory.

## Common Pitfalls

- Assuming the directory creation is successful without checking for errors, which might lead to uncaught errors when the directory already exists or permissions are incorrect.
- Misusing directory permissions, which can lead to security issues or inaccessible directories.
- Forgetting to handle the error if the directory already exists when using `os.Mkdir` instead of `os.MkdirAll`.

## Performance Tips

- Consider checking if the directory exists before attempting to create it to avoid redundant operations.
- Be cautious of creating directories in tight loops without proper checks, which can introduce unnecessary overhead.
- Efficiently manage permissions by setting them correctly during the creation of directories to avoid additional operations afterwards.