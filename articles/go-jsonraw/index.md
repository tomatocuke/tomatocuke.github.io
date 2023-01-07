---
title: "从json.RawMessage学到的东西"
date: "2022-03-18"
categories:
  - Golang
toc: true
---

`json`包中有个`json.RawMessage`其实就是`[]byte`的别名。
其主要是实现了`Marshaler`和`Unmarshaler`两个接口....

<!--more-->
#### 一、应用
`json.RawMessage` 应用在判断情况是否解析，以及解析到什么结构上。例如：
```go
type People struct {
	Age     uint8           `json:"age"`
	Source  uint8           `json:"source"`
	Journey json.RawMessage `json:"journey"`
}
```


情境一：不解析。查看18岁及以上的人，行程里是否去过北京。那么对于`Age < 18` 岁的人就不必解析`Journey`
情景二：解析到异构。国家要统一行程数据，但是之前各省都开发了自己的软件，字段不统一。那汇总的时候`Journey`应该是根据`Source`来源去分别解析到几个不同的结构体上去判别，而那些不需要的字段，不必从里边解析出来。

#### 二、源码
```go
// RawMessage is a raw encoded JSON value.
// It implements Marshaler and Unmarshaler and can
// be used to delay JSON decoding or precompute a JSON encoding.
type RawMessage []byte

// MarshalJSON returns m as the JSON encoding of m.
func (m RawMessage) MarshalJSON() ([]byte, error) {
	if m == nil {
		return []byte("null"), nil
	}
	return m, nil
}

// UnmarshalJSON sets *m to a copy of data.
func (m *RawMessage) UnmarshalJSON(data []byte) error {
	if m == nil {
		return errors.New("json.RawMessage: UnmarshalJSON on nil pointer")
	}
	*m = append((*m)[0:0], data...)
	return nil
}

var _ Marshaler = (*RawMessage)(nil)
var _ Unmarshaler = (*RawMessage)(nil)
```
可以看出`json.RawMessage`是`[]byte`的重定义，分别实现了`Marshaler`接口和`Unmarshaler`接口，在json化的时候什么都不做，在反json化的时候只对源数据拷贝，也不做什么。

##### 我觉得特别值得学习的有两点 ：
- 编译期检测写法。`var _ Marshaler = (*RawMessage)(nil)`  ，断言 `*RawMessage`实现了`Marshaler`接口
- 空间重利用。`*m = append((*m)[0:0], data ...)` 。这个其实大多数情况下没什么用，是说对`RawMessage`重新反Json化的时候，从底层数组原地址开始重写。 举例：
	```go
	type People struct {
		Journey json.RawMessage `json:"journey"`
	}

	func main() {
		var p People

		var bts1 = []byte(`{
			"journey":["杭州","上海"]
		}`)
		var bts2 = []byte(`{
			"journey":["北京"]
		}`)
		var bts3 = []byte(`{
			"journey":["上海","杭州","北京"]
		}`)

		json.Unmarshal(bts1, &p)
		fmt.Printf("%p 初始化地址\n", &p.Journey[0])
		// 第一次解析后记录拷贝一下p.Journey
		a := p.Journey
		fmt.Println(string(a))

		json.Unmarshal(bts2, &p)
		fmt.Printf("%p 在原内存空间上重写，地址不变\n", &p.Journey[0])
		fmt.Println(string(a))

		json.Unmarshal(bts3, &p)
		fmt.Printf("%p 需要扩容，地址变化\n", &p.Journey[0])
		fmt.Println(string(a))

	}
	```
	```shell
	0xc00015e168 初始化地址
	["杭州","上海"]
	0xc00015e168 在原内存空间上重写，地址不变
	["北京"]"上海"]
	0xc000126360 需要扩容，地址变化
	["北京"]"上海"]
	```
	如果不这么写，可以试着把源码复制过来，修改`*m = append((*m)[0:0], data...) ` 为 `*m = append([]byte{}, data...)` ，修改 `Journey json.RawMessage`为 `Journey RawMessage `，可以发现地址一直是变化的，a是不变的。

