---
title: 'Reading, Writing Files'
description: 'Learn how to read from and write to files in Go using the os and io packages'
date: '2025-03-24'
category: 'Files'
---

Go provides robust tools for handling file operations using the `os` and `io` packages. This snippet will guide you through reading from and writing to files in Go.

## Reading Files

Here's a basic way to read a file in Go:

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	data, err := os.ReadFile("example.txt")
	if err != nil {
		log.Fatalf("failed reading file: %s", err)
	}
	fmt.Printf("File contents: %s", data)
}
```

Alternatively, reading a file line by line can be more efficient for large files:

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	f, err := os.Open("example.txt")
	if err != nil {
		log.Fatalf("failed opening file: %s", err)
	}
	defer f.Close()

	scanner := bufio.NewScanner(f)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		log.Fatalf("scanner error: %s", err)
	}
}
```

## Writing Files

Writing data to a file is just as straightforward:

```go
package main

import (
	"log"
	"os"
)

func main() {
	content := []byte("Hello, World!\n")
	err := os.WriteFile("example.txt", content, 0644)
	if err != nil {
		log.Fatalf("failed writing to file: %s", err)
	}
}
```

For appending data, use:

```go
package main

import (
	"log"
	"os"
)

func main() {
	f, err := os.OpenFile("example.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		log.Fatalf("failed opening file: %s", err)
	}
	defer f.Close()

	if _, err := f.WriteString("Appended content\n"); err != nil {
		log.Fatalf("failed writing to file: %s", err)
	}
}
```

## Best Practices

- Always close files with `defer` immediately after opening them
- Use `log` for error handling instead of `panic` in production setups
- Check for scanner errors after scanning is complete
- When writing, handle permissions with the proper file mode (e.g., `0644`)

## Common Pitfalls

- Forgetting to close files, leading to resource leaks
- Not checking for errors and simply assuming file operations succeed
- Using `os.ReadFile` for large files without considering memory constraints
- Not considering file permissions when creating or writing to files

## Performance Tips

- Use `bufio` for reading large files line by line to reduce memory usage
- Preallocate byte slices if you know the file size for performance gains
- Consider using `os.File` methods directly for lower-level file operations
- Avoid using `os.WriteFile` for repeated small writes to reduce system calls