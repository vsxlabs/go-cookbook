---
title: 'Garbage Collection Tuning'
description: 'Optimize garbage collection in Go programs for improved performance'
date: '2025-03-24'
category: 'Performance optimizations'
---

Garbage collection (GC) in Go is a critical feature for automatic memory management. It provides developers with the convenience of not having to manually manage memory, but without careful tuning, GC can also impact the performance of applications. This guide explores several techniques to tune garbage collection in your Go applications.

## Understanding Garbage Collection

Go employs a concurrent and parallel garbage collector that runs automatically, reclaiming memory from objects that are no longer in use. However, GC activity can introduce latency into your application. To optimize this, you can adjust certain runtime parameters.

## Tuning Garbage Collection

### Adjusting GC Target Percentage

The `GOGC` environment variable controls the aggressiveness of the GC. It determines the percentage growth of the heap before a GC cycle is triggered. The default value is 100, which means the heap can grow to 100% of its initial size before a collection run.

```go
package main

import (
	"fmt"
	"runtime/debug"
)

func main() {
	// Set the GC target percentage to 50%.
	debug.SetGCPercent(50)
	
	fmt.Println("Adjusted GOGC to 50%")
	// The GC will now run when the heap grows by 50% instead of the default 100%
}
```

### Monitoring GC Activity

You can monitor GC performance using Go’s built-in metrics. These metrics provide insight into the application’s memory usage and GC behavior.

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	var memStats runtime.MemStats
	runtime.ReadMemStats(&memStats)

	fmt.Printf("Alloc = %v MiB", bToMb(memStats.Alloc))
	fmt.Printf("\tTotalAlloc = %v MiB", bToMb(memStats.TotalAlloc))
	fmt.Printf("\tSys = %v MiB", bToMb(memStats.Sys))
	fmt.Printf("\tNumGC = %v\n", memStats.NumGC)
}

func bToMb(b uint64) uint64 {
	return b / 1024 / 1024
}
```

## Experimental Garbage Collector (Go 1.25+)

Go 1.25 introduces a new experimental garbage collector that significantly improves performance for applications that heavily use garbage collection. The new experimental garbage collector delivers significant performance improvements, with benchmark results showing a 10-40% reduction in garbage collection overhead in real-world programs that heavily use the garbage collector. The design also provides better locality through improved memory locality when processing small objects, and enhanced scalability with better CPU scalability for garbage collection operations.
To use the new experimental garbage collector, set the `GOEXPERIMENT` environment variable at build time:

```bash
GOEXPERIMENT=greenteagc
```

## Best Practices

- **Tune the GOGC Value**: Adjust the `GOGC` value based on your app's performance metrics and memory usage patterns.
- **Profile Regularly**: Use tools like `pprof` to profile and understand the impact of GC on your application.
- **Optimize Memory Usage**: Reduce allocations and deallocate memory promptly if your program allows manual allocations.

## Common Pitfalls

- **Ignoring Memory Leaks**: Not paying attention to memory leaks that continuously grow heap size, resulting in frequent GC cycles.
- **Over-tuning**: Setting `GOGC` too high might reduce GC frequency but will increase memory usage, which can lead to poor performance under memory pressure.

## Performance Tips

- **Minimize Pointer Usage**: Pointers increase the workload of the garbage collector. Reducing unnecessary pointer usage can improve GC performance.
- **Batch Workloads**: Where feasible, handle data in batches to reduce GC overhead.
- **Concurrent Execution**: Utilize goroutines effectively to leverage Go’s concurrency model, allowing other parts of the program to run while the GC executes.

This overview provides foundational techniques for tuning garbage collection in Go applications. Tailor these strategies to your specific use case for optimal performance.