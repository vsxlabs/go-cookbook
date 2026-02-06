---
title: 'Writing and Reading CSV Files'
description: 'Learn how to write to and read from CSV files in Go using the encoding/csv package'
date: '2025-03-24'
category: 'Files'
---

Go provides support for writing to and reading from CSV (Comma-Separated Values) files using the `encoding/csv` package. This guide will demonstrate how to perform these operations efficiently.

## Writing CSV Files

Here's a basic example of how to write data to a CSV file:

```go
package main

import (
	"encoding/csv"
	"os"
	"log"
)

func main() {
	file, err := os.Create("output.csv")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	writer := csv.NewWriter(file)
	defer writer.Flush()

	// Write CSV header.
	header := []string{"Name", "Age", "Email"}
	if err := writer.Write(header); err != nil {
		log.Fatal(err)
	}
	
	// Write CSV records.
	records := [][]string{
		{"John Doe", "30", "john@example.com"},
		{"Jane Smith", "25", "jane@example.com"},
	}
	for _, record := range records {
		if err := writer.Write(record); err != nil {
			log.Fatal(err)
		}
	}
}
```

## Reading CSV Files

To read a CSV file, use the `csv.Reader` like this:

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
	"log"
)

func main() {
	file, err := os.Open("output.csv")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	reader := csv.NewReader(file)

	// Read CSV header.
	headers, err := reader.Read()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Headers:", headers)
	
	// Read CSV records.
	for {
		record, err := reader.Read()
		if err != nil {
			break
		}
		fmt.Println("Record:", record)
	}
}
```

## Handling Different Formats and Configurations

The `csv.Writer` and `csv.Reader` types can be configured to handle various CSV formats:

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
	"log"
)

func main() {
	file, err := os.Open("complex.csv")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	reader := csv.NewReader(file)
	
	// Customize reader options.
	reader.Comma = ';'
	reader.Comment = '#'
	
	// Reading records.
	records, err := reader.ReadAll()
	if err != nil {
		log.Fatal(err)
	}
	for i, record := range records {
		fmt.Printf("Record %d: %v\n", i, record)
	}
}
```

## Best Practices

- Use `defer` to ensure files are properly closed.
- Implement robust error handling for file operations and CSV parsing.
- Use `csv.Writer.Flush()` after writing to ensure all data is written to the file.
- Customize your reader/writer settings based on the specific format of your CSV (e.g., delimiter, comment character).

## Common Pitfalls

- Failing to handle `io.EOF` correctly when reading records.
- Forgetting to close files after operations, potentially leading to resource leaks.
- Ignoring errors from `csv.Writer.Write()` or `csv.Writer.Flush()`, which might result in incomplete writes.
- Assuming all records are of equal length, which might not be true in all CSV files.

## Performance Tips

- Use `csv.Reader.Read()` for line-by-line reading when dealing with large files to minimize memory usage.
- Use `csv.Writer.Write()` in conjunction with `Flush()` to write in batches.
- For performance-critical applications, consider using buffered I/O.
- If you know the number of records in advance, pre-allocate slices to minimize allocations.
- Consider concurrent processing for independent record operations to utilize CPU resources effectively.
