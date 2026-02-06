---
title: 'Building CLI Applications in Go'
description: 'Learn how to build command-line interface (CLI) applications in Go using built-in and external libraries'
date: '2025-03-24'
category: 'CLI Tools'
---

Quick recipes for building CLI applications in Go using different approaches.

## Basic CLI with Flag Package

```go
package main

import (
	"flag"
	"fmt"
	"log"	
)

func main() {
	task := flag.String("task", "", "Task description.")	
	priority := flag.Int("priority", 1, "Priority (1-5).")
	flag.Parse()

	if *task == "" {
		fmt.Println("Error: Task required.")
		log.Fatal("task parameter is required")
	}

	fmt.Printf("Added task: %s (Priority: %d)\n", *task, *priority)
}
```

Run with:
```shell
go run task.go -task="Complete project docs" -priority=3
```

## Subcommands with Cobra

```go
package main

import (
	"fmt"
	"github.com/spf13/cobra"
	"log"
	"os"
)

func main() {
	var priority int

	rootCmd := &cobra.Command{
		Use:   "tasks",
		Short: "Task management CLI.",
	}

	addCmd := &cobra.Command{
		Use:   "add [task]",
		Short: "Add a new task.",
		Args:  cobra.ExactArgs(1),
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Printf("Added task: %s (Priority: %d)\n", args[0], priority)
		},
	}

	addCmd.Flags().IntVarP(&priority, "priority", "p", 1, "Task priority (1-5)")
	rootCmd.AddCommand(addCmd)
	
	if err := rootCmd.Execute(); err != nil {
		log.Fatal(err)
	}
}
```

Run with:
```shell
go run tasks.go add "Review code" -p 2
```

## Quick CLI with urfave/cli

```go
package main

import (
	"fmt"
	"github.com/urfave/cli/v3"
	"log"
	"os"
)

func main() {
	app := &cli.Command{
		Name: "tasks",
		Commands: []*cli.Command{
			{
				Name:  "list",
				Usage: "List all tasks",
				Flags: []cli.Flag{
					&cli.BoolFlag{
						Name:    "completed",
						Aliases: []string{"c"},
						Usage:   "Show completed tasks",
					},
				},
				Action: func(c *cli.Context) error {
					if c.Bool("completed") {
						fmt.Println("Completed tasks:")
					} else {
						fmt.Println("Pending tasks:")
					}
					return nil
				},
			},
		},
	}

	if err := app.Run(os.Args); err != nil {
		log.Fatal(err)
	}
}
```

Run with:
```shell
go run tasks.go list --completed
```

## Best Practices

- Provide user-friendly error messages and application usage information.
- Keep commands modular and use subcommands for better organization.
- Make use of external libraries when advanced features are necessary, but keep dependencies minimal.

## Common Pitfalls

- Not validating user inputs properly, leading to potential runtime errors.
- Ignoring return values or errors, which might cause unhandled exceptions when the application runs.
- Poor command organization leading to a confusing and non-intuitive CLI.

## Performance Tips

- Avoid unnecessary computations in `Run` or equivalent methods to keep the CLI responsive.
- Cache repeated data fetches or computations when possible.
- Profile and optimize the initialization time of your CLI application if start-up performance is an issue.

These strategies and examples demonstrate how Go can be leveraged effectively to create powerful and efficient command-line tools.