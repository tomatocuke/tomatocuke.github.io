---
title: "Go测试chan是否会被自动GC"
date: "2022-12-03"
---

代码为内部技术群大佬写的

```go
package main

import (
	"runtime"
	"unsafe"
)

type markBits struct {
	bytep *uint8
	mask  uint8
	index uintptr
}

//go:linkname isMarked runtime.markBits.isMarked
func isMarked(m markBits) bool

//go:linkname spanOf runtime.spanOf
func spanOf(p uintptr) unsafe.Pointer

//go:linkname objIndex runtime.(*mspan).objIndex
func objIndex(s unsafe.Pointer, p uintptr) uintptr

//go:linkname allocBitsForIndex runtime.(*mspan).allocBitsForIndex
func allocBitsForIndex(s unsafe.Pointer, allocBitIndex uintptr) markBits

func allocBitsForAddr(p uintptr) markBits {
	s := spanOf(p)
	objIndex := objIndex(s, p)
	return allocBitsForIndex(s, objIndex)
}

var ca [10]chan int
var pa [10]uintptr

func main() {
	for i := 0; i < len(pa); i++ {
		c := make(chan int, 10)
		p := *(*uintptr)(unsafe.Pointer(&c))
		c <- 1
		c <- 2
		c <- 3
		ca[i] = c
		pa[i] = p
	}

	for i := 0; i < len(pa); i++ {
		if i%2 == 0 {
			continue
		}
		ca[i] = nil
	}

	runtime.GC()

	for i := 0; i < len(pa); i++ {
		println(isMarked(allocBitsForAddr(pa[i])))
	}
}
```
