---
title: 'Working with Heaps'
description: 'Learn how to utilize heaps in Go using the container/heap package for efficient priority queue management'
date: '2025-03-24'
category: 'Tools'
---

Heaps are a powerful data structure used to implement priority queues efficiently. Go provides a robust package, `container/heap`, that offers heap-based operations. This guide demonstrates how to leverage this package to manage priority queues.

## Basic Heap Implementation

Here's an example of how to create a min-heap using Go's `container/heap` package:

```go
package main

import (
	"container/heap"
	"fmt"
)

// An IntHeap is a min-heap of integers.
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

func main() {
	h := &IntHeap{42, 31, 73}
	heap.Init(h)
	heap.Push(h, 55)
	fmt.Printf("min: %d\n", (*h)[0])
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h))
	}
}
```

## Custom Heap Example

You can define your custom heap by implementing the `heap.Interface`. Here's an example showing a max-heap for managing tasks based on priority:

```go
package main

import (
	"container/heap"
	"fmt"
)

// Task represents a single task with priority.
type Task struct {
	priority int
	name     string
}

// A TaskHeap is a max-heap of tasks based on their priorities.
type TaskHeap []Task

func (h TaskHeap) Len() int           { return len(h) }
func (h TaskHeap) Less(i, j int) bool { return h[i].priority > h[j].priority }
func (h TaskHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *TaskHeap) Push(x interface{}) {
	*h = append(*h, x.(Task))
}

func (h *TaskHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

func main() {
	h := &TaskHeap{
		{priority: 3, name: "Update docs"},
		{priority: 1, name: "Fix bugs"},
		{priority: 2, name: "Add tests"},
	}
	heap.Init(h)
	heap.Push(h, Task{priority: 4, name: "Deploy v2"})
	
	for h.Len() > 0 {
		task := heap.Pop(h).(Task)
		fmt.Printf("Task: %s, Priority: %d\n", task.name, task.priority)
	}
}
```

## Best Practices

- Always call `heap.Init()` before using heap operations to ensure the data structure maintains heap properties.
- Implement both the `Push` and `Pop` methods meticulously to maintain heap consistency.
- Choose `MinHeap` or `MaxHeap` approaches appropriately for your use case, and adjust `Less` method accordingly.

## Common Pitfalls

- Not calling `heap.Init()` before using other heap operations can lead to incorrect results.
- Incorrect implementation of `Less` method can result in an improperly sorted heap.
- Forgetting to manage the return type in `Pop()` when casting `any` back to the original type.

## Performance Tips

- Use heaps for cases where insertions and deletions at specific order priorities are frequent.
- Consider amortized time complexity of O(log n) for `Push` and `Pop`, which is efficient compared to alternative data structures.
- Pre-allocate slice capacity if the heap size is known in advance to optimize memory allocations.