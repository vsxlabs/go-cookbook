---
title: 'Using bufio for Efficient IO'
description: 'Explore how to use the bufio package in Go for efficient I/O operations on large data streams'
date: '2025-03-24'
category: 'Tools'
---

The `bufio` package in Go provides buffered I/O for enhanced performance while reading and writing data. It offers an efficient way to handle large data streams by reducing the number of system calls.

## Basic File Reading with bufio

Read through a file line by line using `bufio.Scanner`:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"log"
)

func main() {
	file, err := os.Open("largefile.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
	
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading file:", err)
	}
}
```

## Efficient File Writing with bufio

Here's how to write data efficiently to a file using `bufio.Writer`:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"log"
)

func main() {
	file, err := os.Create("output.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	_, err = writer.WriteString("Buffered writing with bufio.Writer\n")
	if err != nil {
		log.Fatal(err)
	}

	// Flush buffered data to the file.
	err = writer.Flush()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Successfully written to file.")
}
```

## Configuring bufio.Scanner for Custom Splitting

You can customize the scanner to split input differently:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

func main() {
	input := "Go,bufio,package,example"
	scanner := bufio.NewScanner(strings.NewReader(input))
	
	// Note: ScanWords splits on whitespace, not commas.
	// For comma-separated input, use a custom split function.
	scanner.Split(func(data []byte, atEOF bool) (advance int, token []byte, err error) {
		if atEOF && len(data) == 0 {
			return 0, nil, nil
		}
		if i := strings.IndexByte(string(data), ','); i >= 0 {
			return i + 1, data[0:i], nil
		}
		if atEOF {
			return len(data), data, nil
		}
		return 0, nil, nil
	})

	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}
}
```

## Best Practices

- **Always Close Resources**: Use `defer` to ensure files are closed after their operations are done.
- **Custom Buffer Size**: Use `bufio.NewReaderSize` or `bufio.NewWriterSize` to specify custom buffer sizes when the data pattern suggests non-default sizes.
- **Error Handling**: Always check for errors after operations, especially when using `Flush` on writers.

## Common Pitfalls

- **Scanner Buffer Limitations**: Default token buffer size for `bufio.Scanner` may be inadequate for very large tokens (use `.Buffer()` to adjust).
- **Neglecting Errors**: Failing to handle errors from `Scanner.Err()` can lead to silent failures when reading lines.

## Performance Tips

- **Optimal Buffer Size**: Test different buffer sizes based on your data to achieve optimal performance.
- **Minimize System Calls**: Buffered I/O reduces the frequency of direct system calls, which can significantly enhance performance when processing large files.
- **Batch Processing**: Process data in chunks that fit entirely within the buffer to benefit from the reduced system I/O overhead.

By leveraging the `bufio` package effectively, developers can significantly improve the I/O performance of their Go applications, especially when dealing with large datasets.
