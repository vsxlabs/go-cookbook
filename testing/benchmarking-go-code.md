---
title: 'Benchmarking Go Code'
description: 'Learn how to benchmark your Go code using the testing package to measure performance and optimize accordingly'
date: '2025-03-24'
category: 'Testing'
---

Benchmarking is a crucial practice to measure the performance of your Go code and identify bottlenecks. The Go standard library provides a `testing` package which is well-suited for benchmarking.

## Basic Benchmark Function

Here's a simple example of a benchmark function to measure the performance of a function:

```go
package main

import (
	"testing"
)

// Function to benchmark.
func Sum(x, y int) int {
	return x + y
}

func BenchmarkSum(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = Sum(1, 2)
	}
}
```

To run the benchmark, use the command: `go test -bench=.`

### Go 1.24+ Benchmark with b.Loop Function

You can achieve the same functionality with the `b.Loop` method:

```go
package main

import (
	"testing"
)

// Function to benchmark.
func Sum(x, y int) int {
	return x + y
}

func BenchmarkSum(b *testing.B) {
	for b.Loop() {
		_ = Sum(1, 2)
	}
}
```

This method prevents unwanted compiler optimizations and automatically handles setup and cleanup for you.

## Benchmarking with Setup and Teardown

Often you'll want to include setup or teardown logic with your benchmarks:

```go
package main

import (
	"testing"
)

func ExpensiveSetup() int {
	// Simulate an expensive operation.
	return 42
}

func BenchmarkWithSetupTeardown(b *testing.B) {
	data := ExpensiveSetup()
	b.ResetTimer() // Reset timer after setup

	for i := 0; i < b.N; i++ {
		_ = data + i
	}

	b.StopTimer() // Stop timer before teardown
}
```

## Benchmarking with Parallelism

You can test how well your code performs using parallel execution:

```go
package main

import (
	"testing"
)

func ParallelTask() {
	// Simulate some work.
}

func BenchmarkParallelTask(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			ParallelTask()
		}
	})
}
```

## Best Practices

- Use `b.ResetTimer()` after any setup to ensure only the execution time of the code you're benchmarking is measured.
- Use `b.StopTimer()` and `b.StartTimer()` around parts of the benchmark that shouldn't be timed, like setup and teardown.
- Be mindful of external system effects — ensure controlled conditions to get consistent results.
- Use benchmarks to explore different variations of your code for optimization opportunities.

## Common Pitfalls

- Forgetting to reset the timer (`b.ResetTimer()`) after setup.
- Running benchmarks on a machine with varying loads can result in inconsistent results.
- Failing to ensure benchmark code does meaningful work; avoid empty loops that the compiler might optimize away.
- Not using parallel benchmarks for functions intended to be used concurrently.

## Performance Tips

- Consider using `-benchmem` flag to report memory allocations when running benchmarks.
- Profile your code with `pprof` to find hotspots and memory usage patterns.
- Compare benchmarks before and after changes to evaluate their impact on performance.
- Experiment with different sizes or types of input data to see how they affect performance.
- Use the `b.ReportAllocs()` method to ensure memory allocations are reported.
