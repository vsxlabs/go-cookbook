---
title: 'System Calls in Go'
description: 'Learn how to make system calls in Go using golang.org/x/sys and os/exec packages'
date: '2025-03-24'
category: 'Tools'
---

System calls allow a program to request a service from the kernel of the operating system. In Go, the `syscall` package is deprecated/frozen; use `golang.org/x/sys/...` for low-level system calls or higher-level `os` APIs. For most cases, `os/exec` offers a user-friendly and high-level interface to execute system binaries and commands.

## Using syscall for Low-Level System Calls

The `syscall` package provides low-level interaction with the operating system.

```go
package main

import (
	"fmt"
	"log"
	"syscall"
	"unsafe"
)

func main() {
	// Simple syscall example - get process ID.
	pid, _, _ := syscall.Syscall(syscall.SYS_GETPID, 0, 0, 0)
	
	fmt.Printf("Process ID: %d\n", pid)
	
	// Open a file using syscall.
	fd, _, err := syscall.Syscall(syscall.SYS_OPEN, 
		uintptr(unsafe.Pointer(syscall.StringBytePtr("test.txt"))),
		syscall.O_CREAT|syscall.O_RDWR,
		0644)
	
	if int(err) != 0 {
		log.Fatalf("syscall error: %v", err)
	}
	
	// Don't forget to close the file descriptor.
	syscall.Syscall(syscall.SYS_CLOSE, fd, 0, 0)
}
```

## Using os/exec for High-Level Execution

`os/exec` is commonly used in Go for running external commands, as it provides a straightforward and high-level interface.

```go
package main

import (
	"os/exec"
	"log"
	"fmt"
)

func main() {
	cmd := exec.Command("ls", "-l")
	output, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatalf("command execution failed: %v", err)
	}
	fmt.Printf("Command output: %s\n", output)
}
```

## Executing Commands with Environment

You can also customize the environment when executing commands using `os/exec`.

```go
package main

import (
	"os/exec"
	"log"
	"os"
)

func main() {
	cmd := exec.Command("printenv")
	cmd.Env = append(os.Environ(), "FOO=bar")
	output, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatalf("command execution failed: %v", err)
	}
	log.Printf("Environment: %s\n", output)
}
```

## Best Practices

- Use `os/exec` for executing command-line utilities over `syscall` for simplicity and readability.
- Always consider security implications when executing external commands, avoid using unsanitized input.
- Capture and handle possible errors returned by system calls properly.
- Be aware of differences in system commands across different operating systems.

## Common Pitfalls

- Forgetting to handle both `stdout` and `stderr` when executing commands; use `CombinedOutput()` to capture them together.
- Ignoring the environment needs for the executed command, leading to unexpected behavior.
- Passing incorrect arguments or environment variables can result in failure.
- Not closing any pipes or resources properly when using `os/exec` for more complex setups.

## Performance Tips

- Reuse commands in `os/exec` where possible to prevent repeated costly setup.
- Avoid `syscall` if not necessary, as it involves more overhead in managing low-level OS resources.
- When using `os/exec`, consider using pipes directly for very large data handling to avoid memory limitations.
- Profile your use of system calls to ensure they are not the bottleneck in performance-critical systems.