title: 在Golang中使用json
date: 2015-11-21 15:19:56
tags: go
category: dev
---

包引用
```go
import (
    "encoding/json"
    "github.com/bitly/go-simplejson" // for json get
)
```
用于存放数据的结构体
```go
type MyData struct {
    Name   string    `json:"item"`
    Other  float32   `json:"amount"`
}
```
这里需要注意的就是后面单引号中的内容。

`json:"item"`
这个的作用，就是Name字段在从结构体实例编码到JSON数据格式的时候，使用item作为名字。算是一种重命名的方式吧。

编码JSON
```go
var detail MyData

detail.Name  = "1"
detail.Other = "2"

body, err := json.Marshal(detail)
if err != nil {
    panic(err.Error())
}
```
我们使用Golang自带的encoding/json包对结构体进行编码到JSON数据。

json.Marshal(...)
JSON解码
由于Golang自带的json包处理解码的过程较为复杂，所以这里使用一个第三方的包simplejson进行json数据的解码操作。
```go
js, err := simplejson.NewJson(body)
if err != nil {
    panic(err.Error())
}

fmt.Println(js)
```