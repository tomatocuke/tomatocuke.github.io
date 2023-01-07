---
title: "Golang学习-数据类型"
date: "2019-02-13"
categories:
  - Golang
toc: true
---


<!--more-->

### 一、数字、字符串、常量
```go
func main(){
	fmt.Println("----数字----")
	fmt.Println("不同数字类型不可算数运算，别名可以")
	// 定义类型别名 type New = Old
	// 定义新类型  type New Old

	// 默认的数字类型是int，int不是int32或int64的别名
	n := 123
	fmt.Printf("%T \n", n)

	//uint8(别名"byte",0-255), uint16, uint32, uint64
	var n1 uint8 = 11
	var n2 byte = 22
	// 别名相当于同一类型可以运算
	fmt.Printf("%T %T %T \n", n1, n2, n1+n2)

	//int8, int16, int32(别名"rune"), int(默认), int64
	var n3 rune = 11
	var n4 int32 = 22
	fmt.Printf("%T %T %T \n", n3, n4, n3+n4)

	//float32, float64(默认)
	var f = 3.14
	i := int(f) // 可以强制转换类型
	fmt.Printf("%T %T %d \n", f, i, i)

	// 关于溢出
	var x int8 = -128
	fmt.Println(x - 1)
	fmt.Println(x / -1)
	fmt.Println(x/-1 - 10)
	fmt.Println(x/-1 + 10)


	fmt.Println("----字符串----")
	str := "hallo "
	str += "world"//使用“+”拼接字符串
	//不能直接更改单个字符，拆成字节切片，修改后转回字符串
	s := []byte(str)
	fmt.Println(s)
	s[1] = 'e'
	str = string(s)
	fmt.Println(str)
	//字符串可以切割，字符串切片在底层上并不创建新的内存，是在原有的基础上偏移
	str = "I like the " + str[6:]
	fmt.Println(str)
	// 字符串长度，len()无法计算中文长度，可以利用utf8包，也可以使用[]rune
	str2 := "hello中国"
	fmt.Println(len(str2), len([]rune(str2)))


	fmt.Println("----常量----")
	//特殊常量iota,在 const关键字出现时将被重置为 0,const 中每新增一行常量声明将使 iota 计数一次
	const (
		A = iota
		B
		C string = "C"
		D
		E = iota
	)
	const F = iota
	fmt.Println(A, B, C, D, E, F)
}
```

### 二、指针
```go
package main
import "fmt"
func main() {
   var a int = 4
   //一个指针变量通常缩写为ptr,空指针为nil
   var ptr *int
   fmt.Println(ptr == nil)
   ptr = &a
   fmt.Printf("ptr类型为%T\n", ptr)
   fmt.Println("ptr值为", ptr)
   fmt.Printf("*ptr值为%d\n", *ptr)
}
```
### 三、数组和切片
```go
package main
import (
	"fmt"
)
func main() {
	fmt.Println("----数组----")
	//数组是具有相同唯一类型的一组已编号且长度固定的数据项序列
	arr1 := [3]byte{'a','b'}
	//也可以自行统计[...]byte{'a','b'}，但此时固定长度为2，则不能进行下边赋值
	arr1[2] = 'c'
	//二维数组
	arr2 := [2][2]int{{1,2},{3}}
	arr2[1][1] = 4
	fmt.Println(arr1, arr2)


	fmt.Println("----切片----")
	//切片("动态数组",引用类型),与数组相比切片的长度是不固定的，可以追加元素
	//像一个结构体，包含了指针，长度，最大长度
	slice1 := []int{1,2}
	slice2 := append(slice1, 3)
	slice3 := slice2[1:]
	fmt.Println(slice1,slice2,slice3)
	fmt.Println(cap(slice1))
	//使用make形式,第二个参数代表初始化2个默认成员，第三个参数表示最大成员数（可省略）。
	slice4 := make([]int, 2, 3)//2个成员
	slice5 := append(slice4, 67373)//3个成员，地址和slice4相同
	slice6 := append(slice5, 67373)//超出，重新分配地址
	fmt.Println(len(slice4),cap(slice4),slice4,slice5)
	fmt.Printf("%p %p %p \n",slice4,slice5, slice6)
	//未指定长度的切片等于nil
	var slice7 []int
	fmt.Println(slice7 == nil)
}
```
切片cap容量增长原理：append函数执行时会判断切片容量是否能够存放新增元素，如果不能，则会重新申请存储空间，新存储空间将是原来的2倍或1.25倍
```go
package main

import (
	"fmt"
)

func main() {
	var slice []int
	slice = append(slice, 1)
	fmt.Println(&slice[0], cap(slice))

	slice = append(slice, 2)
	fmt.Println(&slice[0], cap(slice))

	slice = append(slice, 3)
	fmt.Println(&slice[0], cap(slice))

	slice = append(slice, 4)
	fmt.Println(&slice[0], cap(slice))

	slice = append(slice, 5)
	fmt.Println(&slice[0], cap(slice))

	for i := 0; i < 256; i++ {
		slice = append(slice, i)
	}
	fmt.Println(&slice[0], cap(slice))

	for i := 0; i < 256; i++ {
		slice = append(slice, i)
	}
	fmt.Println(&slice[0], cap(slice))

	for i := 0; i < 256; i++ {
		slice = append(slice, i)
	}
	fmt.Println(&slice[0], cap(slice))

	for i := 0; i < 256; i++ {
		slice = append(slice, i)
	}
	fmt.Println(&slice[0], cap(slice))

}

```

### 四、Map集合
```go
package main
import "fmt"
func main() {
	// 声明map时，是个空指针
	var m0 map[int]int
	// m0[0] = 0 不可以
	fmt.Println(m0, m0 == nil)
	
	//初始化, 正确姿势是使用make分配内存
	m1 := map[string]int{"a": 1}
	//添加键
	key := "b"
	m1[key] = 2
	//map为引用类型，m1和m2同一个地址
	m2 := m1
	m2["c"] = 3
	fmt.Println(m1)
	//遍历，注意是无序的！随机的！
	for key, val := range m1 {
		fmt.Printf("key=%s,val=%d\n", key, val)
	}
	//删除，不存在不会报错
	delete(m1, "c")
	fmt.Println(m2)
}
```
### 五、Struct结构体
```go
package main
import "fmt"
type Human struct{
	Name string
	Age int
}
//组合，相当于类继承
type Student struct{
	Human
	Number int
}
func main(){
	me := Student{
		Human : Human{ Name: "abc"},
		Number: 67373,//换行结束时此处必须有逗号
	}
	//两种形式皆可获得，未赋值Age数字类型默认为0
	me.Age = 1
	me.Human.Age = 2
	fmt.Println(me)
	
	// 可以使用 new 去创建，但是返回的是指针类型
	you := new(Student)
	you.Number = 18 //指针类型不能.属性，这里做了语法糖，其实是(*you).Number 
	fmt.Printf("%T\n", you)

	//匿名结构体，临时使用
	his := struct{
		name string
		age uint8
	}{"his", 34} // 可以不指定key，值列表形式
	fmt.Println(his)

}
```
### 六、函数
```go
package main
import "fmt"

//多个返回值
func fn1(c int) (d int,e int){
	d,e = c + 1, c + 2
	return 
}
func fn2(c int) (int, int){
	return c + 1, c + 2
}
//不定参数生成反转数组
func fn3(arg ...int) []int{
	length := len(arg)
	s := make([]int, length)
	for i := 0; i < length; i++ {
		s[i] = arg[length - i - 1]
	}
	return s
}

// 专有函数,也就是方法，作用于特定类型，需要有接收者
// 举例先声明一个结构体
type Me struct{
	name string
}
// 这里的变量名应使用结构体首字母小写，不要用this或者self
func (m *Me) say () {
	fmt.Println("我是" + m.name)
}

func main() {
	c := 1
	//多返回值
	d,e := fn1(c)
	f,g := fn2(c)
	fmt.Println(d, e, f, g)
	//不定参数
	s := fn3(1,2,3,4)
	fmt.Println(s)
	//say不能直接调用
	me := Me{"张三"}//new(Me)
	me.say()
}
```
### 七、接口
```go
package main
import "fmt"
func main() {
	//空接口可以传任何类型
	type Element interface{}
	//一个空接口类型的切片的值可以是任意类型
	list := []Element{1,"Hello",'e'}
	fmt.Println(list)
	//断言类型
	val,ok := list[0].(int)
	fmt.Println(val,ok)
	
	//使用接口类型
	var phone Phone = new(Huawei)//这里必须取址,等同&Huawei{}
	phone.call()
}

//定义接口
type Phone interface {
	call()
}
//定义结构体
type Huawei struct {

}
//接口实现方法
func (huawei *Huawei) call() {
	fmt.Println("我用华为打电话")
}
```
### 八、并发goruntine
```go
package main 
import (
	"fmt"
	"runtime"
)
func main() {
	fmt.Printf("CPU数量%d \n",runtime.NumCPU())  // 注意，这里返回的是逻辑核数，非物理核心数
	runtime.GOMAXPROCS(2)//设置可以并行计算的CPU核数，返回之前的值

	for i := 0; i < 10; i++ {
		//固定顺序
		fmt.Println(i)
		//非固定顺序，且不一定有机会输出，程序就结束了
		go func(i int){
			fmt.Printf("goruntine%d\n", i)
		}(i)
	}
	
}
```
使用 ``sync.WaitGroup``保证goroutine都正常执行
```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	fmt.Printf("CPU数量%d \n", runtime.NumCPU()) // 注意，这里返回的是逻辑核数，非物理核心数
	runtime.GOMAXPROCS(2)                      //设置可以并行计算的CPU核数，返回之前的值

	wg := sync.WaitGroup{}

	for i := 0; i < 10; i++ {
		wg.Add(1) // +1
		//固定顺序
		fmt.Println(i)
		//非固定顺序，且不一定有机会输出，程序就结束了
		go func(i int) {
			fmt.Printf("goruntine%d\n", i)
			wg.Done() // -1
		}(i)
	}

	wg.Wait() // 等待
}
```
### 九、chan通道
```go
package main
import (
    "fmt"
	"time"
)
//通道可以声明接收或者发送类型
// type Sender chan<- int
// type Receiver <-chan int
func main() {
	//默认非缓存阻塞的，即第二个参数默认0.
	//设置缓存数量，可以直接写入，（再直接写入会报错），其后的可以通过goruntine写入
	c := make(chan int, 2)
	c <- 1
	c <- 2
	go func(c chan<- int) {
		for i := 3; i < 6; i++ {
			c <- i
			fmt.Println("发送",i)
		}
		close(c)
	}(c)
	//1,2可以直接读取，goruntine不一定这时候有没有发送，所以此时等待，接收到便读取，直到close
	for i := range c {
		fmt.Println("接收", i)
	}
	// 让main函数执行结束的时间延迟1秒，不然goroutine大概率没机会输出。
	// 以使上面两个代码块有机会被执行。这不是正确姿势，应该使用sync.WaitGroup包，或者通过非缓存阻塞的
	time.Sleep(time.Second)
}
```
非缓存阻塞的channel，select用于监听IO
```go
package main

import (
	"fmt"
	"time"
)

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)

	ticker := time.NewTicker(time.Millisecond * 500)
	go func() {
		for t := range ticker.C {
			fmt.Println("Tick at", t)
		}
	}()
}

```

理解 select ， 可以再尝试去掉 c = nil 查看一下输出。 （这个range空对象数组妙啊）
```go
var o = fmt.Print

func main() {
    c := make(chan int, 1)
    for range [4]struct{}{} {
        select {
        default:
            o(1)
        case <-c:
            o(2)
            c = nil
        case c <- 1:
            o(3)
        }
    }
}
```
