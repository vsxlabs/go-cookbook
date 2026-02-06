---
title: 'Writing and Reading XML Files'
description: 'Learn how to write to and read from XML files in Go using the encoding/xml package'
date: '2025-03-24'
category: 'Files'
---

Go provides support for writing to and reading from XML files using the `encoding/xml` package. This guide will demonstrate how to perform these operations efficiently.

## Writing XML Files

Here's a basic example of how to write data to an XML file:

```go
package main

import (
	"encoding/xml"
	"os"
	"log"
)

type Person struct {
	XMLName   xml.Name `xml:"person"`
	Name      string   `xml:"name"`
	Age       int      `xml:"age"`
	Email     string   `xml:"email"`
}

type People struct {
	XMLName xml.Name `xml:"people"`
	Persons []Person `xml:"person"`
}

func main() {
	file, err := os.Create("output.xml")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	people := People{
		Persons: []Person{
			{Name: "John Doe", Age: 30, Email: "john@example.com"},
			{Name: "Jane Smith", Age: 25, Email: "jane@example.com"},
		},
	}

	encoder := xml.NewEncoder(file)
	encoder.Indent("", "  ")
	
	// Write XML header
	file.WriteString(xml.Header)
	
	// Encode the data to XML
	if err := encoder.Encode(people); err != nil {
		log.Fatal(err)
	}
}
```

## Reading XML Files

To read an XML file, use the `xml.Decoder` like this:

```go
package main

import (
	"encoding/xml"
	"fmt"
	"os"
	"log"
)

type Person struct {
	XMLName xml.Name `xml:"person"`
	Name    string   `xml:"name"`
	Age     int      `xml:"age"`
	Email   string   `xml:"email"`
}

type People struct {
	XMLName xml.Name `xml:"people"`
	Persons []Person `xml:"person"`
}

func main() {
	file, err := os.Open("output.xml")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	decoder := xml.NewDecoder(file)
	var people People
	
	if err := decoder.Decode(&people); err != nil {
		log.Fatal(err)
	}
	
	fmt.Println("Number of people:", len(people.Persons))
	for i, person := range people.Persons {
		fmt.Printf("Person %d: %s, %d years old, %s\n", 
			i, person.Name, person.Age, person.Email)
	}
}
```

## Handling Complex XML Structures

Go's `encoding/xml` package can handle complex XML documents with nested elements and attributes:

```go
package main

import (
	"encoding/xml"
	"fmt"
	"os"
	"log"
)

type Address struct {
	Street  string `xml:"street"`
	City    string `xml:"city"`
	Country string `xml:"country"`
}

type Person struct {
	XMLName   xml.Name  `xml:"person"`
	ID        int       `xml:"id,attr"`
	Name      string    `xml:"name"`
	BirthDate string    `xml:"birthDate"`
	Address   Address   `xml:"address"`
	Notes     string    `xml:",cdata"`
	Active    bool      `xml:"active"`
}

func main() {
	file, err := os.Open("complex.xml")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	decoder := xml.NewDecoder(file)
	
	var person Person
	if err := decoder.Decode(&person); err != nil {
		log.Fatal(err)
	}
	
	fmt.Printf("Person: %s (ID: %d)\n", person.Name, person.ID)
	fmt.Printf("Address: %s, %s, %s\n", 
		person.Address.Street, person.Address.City, person.Address.Country)
}
```

## Best Practices

- Use `defer` to ensure files are properly closed.
- For large files, consider using `Decoder.Token()` to process XML documents incrementally.
- Correctly handle namespaces when working with more complex XML documents.

## Common Pitfalls

- Forgetting to handle XML namespaces properly.
- Not using appropriate struct tags for customizing XML output.
- Missing error handling during encoding/decoding operations.
- Incorrectly structuring Go types to match the XML structure.
- Memory issues when loading large XML documents entirely into memory.

## Performance Tips

- Use streaming via `Decoder.Token()` for processing large XML files to reduce memory usage.
- Consider using buffered I/O for better performance.
- Pre-allocate slices if you know the approximate size of collections in advance.
- For large documents, consider SAX-style parsing instead of DOM-style parsing.
- When possible, use more specific types rather than interfaces to avoid type assertions.
