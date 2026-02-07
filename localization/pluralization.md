---
title: 'Pluralization in Localization'
description: 'Learn how to handle pluralization in internationalized Go applications, ensuring accurate translations across languages.'
date: '2025-03-24'
category: 'Localization'
---

Handling pluralization in localization is essential for creating applications that support multiple languages. Different languages have unique rules for pluralization, making it crucial to implement a flexible solution. Below, we'll explore techniques and libraries to manage pluralization in Go applications.

## Basic Example Using `text/template`

Go's `text/template` package can be used to create basic, language-agnostic pluralization logic. Here's an example without any localization libraries:

```go
package main

import (
	"fmt"
	"text/template"
	"os"
)

func main() {
	tmpl := `{{.Count}} {{if eq .Count 1}}{{.Singular}}{{else}}{{.Plural}}{{end}}`
	data := []struct {
		Count int
		Singular, Plural string
	}{
		{1, "apple", "apples"},
		{2, "apple", "apples"},
	}

	for _, d := range data {
		t := template.Must(template.New("pluralize").Parse(tmpl))
		t.Execute(os.Stdout, d)
		fmt.Println()
	}
}
```

## Using `go-i18n` for Pluralization

Popular Go localization library `go-i18n` supports pluralization in a more comprehensive way, using message catalogs. Here's how it can be done:

1. **Importing `go-i18n`:**

   First, you need to install the package:

   ```
   go get github.com/nicksnyder/go-i18n/v2/i18n
   go get github.com/nicksnyder/go-i18n/v2/i18n/i18nmessage
   ```

2. **Create translation files** (in JSON format for simplicity):

   *active.en.json*

   ```json
   {
     "apple": {
       "one": "You have 1 apple.",
       "other": "You have {{.Count}} apples."
     }
   }
   ```

3. **Implementing Go code:**

   ```go
   package main

   import (
       "encoding/json"
       "fmt"
       "github.com/nicksnyder/go-i18n/v2/i18n"
       "golang.org/x/text/language"
   )

   func main() {
       bundle := &i18n.Bundle{DefaultLanguage: language.English}
       
       bundle.RegisterUnmarshalFunc("json", json.Unmarshal)
       bundle.MustLoadMessageFile("active.en.json")

       localizer := i18n.NewLocalizer(bundle, language.English.String())
       
       messages := []int{1, 3}

       for _, count := range messages {
           msg := localizer.MustLocalize(&i18n.LocalizeConfig{
               MessageID: "apple",
               PluralCount: count,
           })
           fmt.Println(msg)
       }
   }
   ```

## Best Practices

- **Utilize Established Libraries:** Leverage libraries like `go-i18n`, which provide built-in support for pluralization and other localization needs.
- **Understand Locale Differences:** Pluralization rules can vary significantly; always refer to the specific locale settings.
- **Fallback Messages:** Implement fallback messages or values to handle cases where translations might be missing.

## Common Pitfalls

- **Ignoring Plural Rules:** Assuming a single pluralization rule fits all languages.
- **Missing Plural Forms:** Not accounting for all required plural forms for a given language.
- **Hard-Coded Plural Logic:** Avoid embedding pluralization logic within the application code that is difficult to adjust for locale-specific rules.

## Performance Tips

- **Message Bundle Optimization:** Only load what is necessary for the supported languages to reduce memory usage.
- **Cache Localized Messages:** Cache frequently accessed localized messages to minimize lookup time.
- **Lazy Loading:** Implement lazy loading for translation files to improve the application start-up time, especially when supporting multiple languages.