---
title: "Go内存逃逸测试"
date: "2021-11-09"
categories:
  - Golang
toc: true
---


<!--more-->

### 代码
```go
package main

import (
	"fmt"
	"time"
)

type TM1 struct {
	array [1024 * 64]byte
}

type TM2 struct {
	array [1024*64 + 1]byte
}

func TestTM1() {
	// 仅生成结构体
	startTime := time.Now()
	for i := 0; i < 10000; i++ {
		s := TM1{}
		s.array[0] = 1
	}
	fmt.Println("等于64KB结构体：", time.Since(startTime))

	// 生成结构体指针
	startTime = time.Now()
	for i := 0; i < 10000; i++ {
		s := new(TM1)
		s.array[0] = 1
	}
	fmt.Println("等于64KB结构体指针：", time.Since(startTime))
}

func TestTM2() {
	startTime := time.Now()
	for i := 0; i < 10000; i++ {
		s := TM2{}
		s.array[0] = 1
	}
	fmt.Println("大于64KB结构体：", time.Since(startTime))
	startTime = time.Now()
	for i := 0; i < 10000; i++ {
		s := new(TM2)
		s.array[0] = 1
	}
	fmt.Println("大于64KB结构体指针：", time.Since(startTime))
}

func main() {
	TestTM1()
	TestTM2()
}

```
### 结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/8fc910fa01034291b03268e5c768a1c4.png)

### 结论
堆栈分配的效率差距很大，在64位机器上64KB 是一个节点。
TM1 单结构体大小=64KB，无论是结构体还是结构体指针都在栈中生成。
TM2 单结构体大小>64KB，结构体在栈中生成，结构体指针在堆中生成。


### 但是
以上结论过于肤浅，存在内存逃逸这件事。
可以多试2次，分别在TM1  增加 ``fmt.Println(s)``、``fmt.Println(&s)`` ，不要运行 !!! 太长了，使用``go build -gcflags '-m -l' 文件``查看是否存在``escapes to heap`` 或 ``moved to heap``。
更多关于内存逃逸可以参考 [https://zhuanlan.zhihu.com/p/91559562](https://zhuanlan.zhihu.com/p/91559562)  和 [https://geektutu.com/post/hpg-escape-analysis.html](https://geektutu.com/post/hpg-escape-analysis.html) (这篇中有个错误，等于64KB是不会逃逸的，可能是go版本原因)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bea367e67c5043dd82085756562b4b56.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ZOq5ZCS55qE5bCP6Lef54-t,size_20,color_FFFFFF,t_70,g_se,x_16)
补充： ``interface{}`` 动态类型逃逸

所以：一般情况下，对于需要修改原对象值，或占用内存比较大的结构体，选择传指针。对于只读的占用内存较小的结构体，直接传值能够获得更好的性能。
