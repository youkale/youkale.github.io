---
title: url.Values转换struct
date: 2018-01-13 15:36:00
tag: golang
---

### 前言

大家在写golang http服务的时候或许会碰到Request中url.Values转换成struct的需要。

### 思路

翻开net.url查看url.Values的定义

```bash
type Values map[string][]string
```

> 那么我是不是可以通过遍历struct的Field获取对应的数据类型，以及通过tag来从url.Values中获取对应的参数？
> 答案是可以的，那么我们就开动吧。

- 先来定义一个struct,还有一个叫param的tag。

```bash
type User struct {
     UserId int  `param:"user_id,100"
}
```

- struct说明

```bash
字段名:  UserId
url.Values中的字段名:  user_id
默认值: 100
```

- 实现

```bash
    typ := val.Type()
	for i := 0; i < val.NumField(); i++ {
		kt := typ.Field(i) //获取字段类型
		tag := kt.Tag.Get("param") //获取tag
                sv := val.Field(i) //获取字段值
		uv := getVal(values, tag) //获取默认值
               switch sv.Kind() {
		case reflect.String:
			....
		case reflect.Bool:
		        ....
		}
}
```

### 性能测试

```
goos: linux
goarch: amd64
pkg: github.com/youkale/go-querystruct/params
2000000000	         0.00 ns/op
PASS
```

### 最后

好了，思路基本上是这样的,具体实现细节请参考

[GitHub源码地址](https://github.com/youkale/go-querystruct) 
