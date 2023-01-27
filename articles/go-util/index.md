---
title: "Go中一些工具函数封装"
date: "2022-07-13"
---

### 类型转换
- `int`和`string`
```go
func Int2String(i int) string {
  return strconv.Itoa(i)
}

func String2Int(s string) int {
  i, err := strconv.Itoa(s)
  if err != nil {
    fmt.Println(err)
  }
  return i
}
```
  
- `string`和`[]byte`。利用指针直接构造的方式，可以避免拷贝开销，对大字符串有很好的效果。
```go
// 小于1.20
func String2Bytes(s string) []byte {
	bs := struct {
		string
		Cap int
	}{s, len(s)}
	return *(*[]byte)(unsafe.Pointer(&bs))
}

func Bytes2String(bs []byte) string {
	return *(*string)(unsafe.Pointer(&bs))
}

// 1.20
func String2Bytes(s string) []byte {
	return unsafe.Slice(unsafe.StringData(s), len(s))
}

func Bytes2String(bs []byte) string {
	return unsafe.String(&bs[0], len(bs))
}
```


- `string`和`float`
```go
func Float2String(f float64) string {
	return strconv.FormatFloat(f, 'f', -1, 64)
}

func String2Float(s string) float64 {
	f, err := strconv.ParseFloat(s, 64)
  if err != nil {
    fmt.Println(err)
  }
  return f
}
```

- IPV4地址转`int`，IPV4其实本来就是4字节长度，便于查看才有点分十进制的展示形式，为了节省空间，数据库存储最后还是使用`int`
```go
func Ip2Int(ip string) int {
	i := big.NewInt(0).SetBytes(net.ParseIP(ip).To4()).Int64()
	return int(i)
}

func Int2Ip(ip int) string {
	return net.IPv4(byte(ip>>24), byte(ip>>16), byte(ip>>8), byte(ip)).String()
}
```

- 结构体转`map`，在刚开始写go的时候会有这种需求，是认识不足的表现，这是没必要且低效的，并不建议使用。不够可以参考这个看看反射。
```go 
// @structPtr 结构体指针
// @ignoreNil 是否忽略空指针字段
func Struct2Map(structPtr interface{}, ignoreNil bool) map[string]interface{} {
	obj := reflect.ValueOf(structPtr)
	// 不是结构体指针直接返回
	if obj.Kind() != reflect.Ptr || obj.Elem().Kind() != reflect.Struct {
		return nil
	}
	v := obj.Elem()
	t := v.Type()
	n := v.NumField()

	myMap := make(map[string]interface{})
	for i := 0; i < n; i++ {
		itemVal := v.Field(i)
		isPtr := itemVal.Kind() == reflect.Ptr
		// 忽略空指针
		if ignoreNil && isPtr && itemVal.IsNil() {
			continue
		}
		// 需要有json标签
		key := t.Field(i).Tag.Get("json")
		if key == "" || key == "-" {
			continue
		}

		myMap[key] = itemVal.Interface()
	}
	return myMap
}
```


### 随机数

```go
// 预定义字符集
const (
	Number      = "0123456789"
	LowerLetter = "abcdefghijklmnopqrstuvwxyz"
	UpperLetter = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
)

// 随机种子。这种方式为在初始化时固定随机种子
func init() {
	rand.Seed(time.Now().UnixNano())
}

// 随机数
func RandInt(max int) int {
	return rand.Intn(max)
}

// 随机字符串：数字+大写字母
func RandString(length int) string {
	return RandStringWithCharset(length, Number+UpperLetter)
}

// 随机字符串：用户自定义字符集
func RandStringWithCharset(length int, letter string) string {
	b := make([]byte, length)
	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	n := len(letter)
	for i := range b {
		b[i] = letter[r.Intn(n)]
	}
	return *(*string)(unsafe.Pointer(&b))
}
```


### 时间
```go 
// go1.20新增以下三个常量
const (
	DateTime = "2006-01-02 15:04:05"
	DateOnly = "2006-01-02"
	TimeOnly = "15:04:05"
)

func Now() string {
	return time.Now().Format(DateTime)
}

func DayStart(add int) time.Time {
	y, m, d := time.Now().Date()
	return time.Date(y, m, d+add, 0, 0, 0, 0, time.Local)
}

func DayEnd(add int) time.Time {
	y, m, d := time.Now().Date()
	return time.Date(y, m, d+add, 23, 59, 59, 0, time.Local)
}

func MonthStart(add int) time.Time {
	y, m, _ := time.Now().Date()
	return time.Date(y, m+time.Month(add), 1, 0, 0, 0, 0, time.Local)
}

func MonthEnd(add int) time.Time {
	y, m, _ := time.Now().Date()
	return time.Date(y, m+time.Month(add)+1, 1, 0, 0, 0, 0, time.Local).Add(-1 * time.Second)
}

```


### 文件
```go 
// 判断文件是否存在
func IsFileExist(name string) bool {
	_, err := os.Stat(name)
	return err == nil || !os.IsNotExist(err)
}

// 打开文件追加写。（自动创建目录和文件）
func OpenFile(name string) (*os.File, error) {
	if !IsFileExist(name) {
		dir := filepath.Dir(name)
		if !IsFileExist(dir) {
			err := os.MkdirAll(dir, os.ModePerm)
			if err != nil {
				return nil, err
			}
		}
	}
	f, err := os.OpenFile(name, os.O_RDWR|os.O_APPEND|os.O_CREATE, os.ModePerm)
	return f, err
}

```

### http
```go 

var (
	// 默认超时时间
	defaultTimeout = time.Second * 3
)

type Http struct {
	request
	response
}

type request struct {
	Url     string            `json:"url"`
	Method  string            `json:"method"`
	Timeout time.Duration     `json:"-"`
	Header  map[string]string `json:"-"`
	Params  string            `json:"params"`
	Body    []byte            `json:"-"`
}

type response struct {
	Cost   uint8           `json:"cost,omitempty"`
	Err    error           `json:"error,omitempty"`
	Status int             `json:"status"`
	Body   json.RawMessage `json:"body"`
}

// Demo: pkg.HttpGet("http://xxx.com/yy", map[string]string{"a": "1"}).Do()
func HttpGet(url string, params map[string]string) *Http {
	var h Http
	h.Method = http.MethodGet
	if len(params) > 0 {
		h.Url = url + "?" + string(h.buildQuery(params))
	} else {
		h.Url = url
	}
	return &h
}

func HttpPostJson(url string, bodyData interface{}) *Http {
	var h Http
	h.Method = http.MethodPost
	h.Url = url
	h.Header = map[string]string{"Content-Type": "application/json"}
	h.request.Body, _ = json.Marshal(bodyData)
	return &h
}

func HttpPostForm(url string, bodyData map[string]string) *Http {
	var h Http
	h.Method = http.MethodPost
	h.Url = url
	h.Header = map[string]string{"Content-Type": "application/x-www-form-urlencoded"}
	h.request.Body = h.buildQuery(bodyData)
	return &h
}

func (h *Http) SetTimeout(timeout time.Duration) *Http {
	h.Timeout = timeout
	return h
}

func (h *Http) AddHeader(key, val string) *Http {
	if len(h.Header) == 0 {
		h.Header = make(map[string]string)
	}
	h.Header[key] = val
	return h
}

func (h *Http) Do() ([]byte, error) {
	var (
		req    *http.Request
		client *http.Client
		resp   *http.Response
	)
	defer h.printLog(time.Now())

	req, h.Err = http.NewRequest(h.Method, h.Url, bytes.NewReader(h.request.Body))
	if h.Err != nil {
		return nil, h.Err
	}

	for k, v := range h.Header {
		req.Header.Set(k, v)
	}
	if h.Timeout == 0 {
		h.Timeout = defaultTimeout
	}

	// 发送请求
	client = &http.Client{Timeout: h.Timeout}
	resp, h.Err = client.Do(req)
	if h.Err != nil {
		return nil, h.Err
	}
	defer resp.Body.Close()

	h.response.Status = resp.StatusCode
	if resp.StatusCode != http.StatusOK {
		return nil, errors.New("")
	}

	h.response.Body, _ = io.ReadAll(resp.Body)
	return h.response.Body, h.Err
}

func (h *Http) IsOK() bool {
	return h.Status == http.StatusOK
}

func (h *Http) buildQuery(params map[string]string) []byte {
	if len(params) == 0 {
		return nil
	}

	buffer := &bytes.Buffer{}
	var i int
	for k := range params {
		buffer.WriteString(k)
		buffer.WriteByte('=')
		buffer.WriteString(params[k])
		if i != len(params)-1 {
			buffer.WriteByte('&')
		}
		i++
	}

	return buffer.Bytes()
}

func (h *Http) printLog(start time.Time) {
	h.Cost = uint8(time.Since(start).Seconds())
	bs, _ := json.Marshal(h)
	log.Writer().Write(bs)
}
```
