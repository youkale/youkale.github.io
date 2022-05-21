+++
draft = false
date = 2022-05-21T12:32:29+08:00
title = "如何设计RESTful API"
description = "rest API常见设计错误"
slug = ""
authors = []
tags = ["rest"]
categories = []
externalLink = ""
series = []
+++

## 什么是REST 

Wiki中REST全称是 (**Representational State Transfer**) "表现层状态转换", 也有翻译是“表征状态转移”，是[Roy Thomas Fielding](https://en.wikipedia.org/wiki/Representational_state_transfer)博士在2000年提出的一种基于Http协议的软件架构风格。**表征状态转移**,听起来很晦涩，是因为前面主语 `Resource` 被去掉了，真正的全称是 **Resource Representational State Transfer** 也就是: 资源在网络中以某种表现形式进行状态转移。

- Resource：资源，即数据（前面说过网络的核心）。比如 users，corp等；
- Representational：某种表现形式，比如用JSON，XML，JPEG等；
- State Transfer：状态变化。通过HTTP动词实现。



## [REST等级](https://martinfowler.com/articles/richardsonMaturityModel.html#level0)

### Level-0 The Ramp of Plain Old XML 

##### 说明 

服务端使用统一的请求路径，通过请求体中某个字段定义业务类型，需要定义错误码。

#### 调用

对照文档中``` action``` 类型进行传参, 同时对响应错误码进行处理。

```shell
POST /server 
Content-Type: Application/json
{
   "action":"001",
   "data": {
   		"username":"...",
   }
}

----- Response -----

Status: 200
Content-Type: application/json
{
   "status": "A00001"
   "data": {
   			....
   }
}
```

### Level-1 Resource: 引入资源的RPC

#### 说明 

在微服务盛行的今天，引入了URI，方便网关层对请求进行转发，同时能通过URI能反映出对应的业务，需要定义错误码。

#### 调用

对照文档中资源动作，进行传递参数，同时对响应错误码进行处理。

```shell
POST /user/add 
Content-Type: Application/json
{
   "username":""
}

----- Response -----

Status: 200
Content-Type: application/json
{
   "status": "A00001"
   "data": {
   			....
   }
}
```

### Level-2 Http Verbs

#### 说明

- 通过URI定位资源
- 通过Http Method管理资源
- 通过Http Status 来表示结果

#### 调用

查看资源地址，对于有副作用的操作，需要查看文档。

```shell
POST /users
Content-Type: Application/json
{
   "username":""
}
----- Response -----

Status: 201
Location: /users/1
.....
.....
```

```shell
GET /users/1
----- Response -----

Status: 200
Content-Type: Application/json
{
    "username":"...."
}
```

```shell
DELETE /users/1
----- Response -----
Status: 202
....
....
```

### Level-3 Hypermedia Controls : 超媒体控制

#### 说明：

REST的设计者Roy T. Fielding在博客 [REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven) 中的几点原则强调非HATEOAS的系统不能称为RESTful。 

1. REST API决不能定义固定的资源名称或者层次关系

2. 使用REST API应该只需要知道初始URI（书签）和一系列针对目标用户的标准媒体类型

   > 所有的链接都有服务端控制，即服务端改变了URI，客户端也不会被破坏

上面仅仅提到的是HATEOAS对REST的一些约束但是并没有说到具体的规范。好在现在已经出了几个基于HATEOAS的实现:

- HAL: Hypetext Application Language. [IETF草案](https://datatracker.ietf.org/doc/html/draft-kelly-json-hal-08) 
  - [Quarkus 实现](https://quarkus.io/guides/rest-data-panache)  
  - [Spring 实现](https://spring.io/projects/spring-hateoas)  

- [Siren](https://github.com/kevinswiber/siren)
- [Collection](http://amundsen.com/media-types/collection/)
- [JSON-LD](http://json-ld.org/)



## HAL规范

HAL文档使用media type为 hal+json, 在http请求中为: ```Content-Type: application/hal+json```。其ROOT对象必须是一个资源对象。

例如:

```shell
GET /orders/523 HTTP/1.1
Host: example.org
Accept: application/hal+json

----- Response -----

HTTP/1.1 200 OK
Content-Type: application/hal+json
{
  "_links": {
    "self": {
      "href": "/orders/523"
    },
    "warehouse": {
      "href": "/warehouse/56"
    },
    "invoice": {
      "href": "/invoices/873"
    }
  },
  "currency": "USD",
  "status": "shipped",
  "total": 10.2
}
```

### 资源对象

一个资源对象代表一个资源。

它有两个保留属性

_links: 包含一个指向其他资源的链接

_embedded: 可选属性， 它是一个对象，其属性名称是链接关系类型，其值是一个资源对象或一个资源对象数组。资源对象的数组。嵌入的资源可以是完整的、部分的或不一致的版本。嵌入资源可能是目标URI提供的完整、部分或不一致的表示。

### 举个例子

```json
{
  "_links": {
    "self": {
      "href": "/orders"
    },
    "next": {
      "href": "/orders?page=2"
    },
    "find": {
      "href": "/orders{?id}",
      "templated": true
    }
  },
  "_embedded": {
    "orders": [
      {
        "_links": {
          "self": {
            "href": "/orders/123"
          },
          "basket": {
            "href": "/baskets/98712"
          },
          "customer": {
            "href": "/customers/7809"
          }
        },
        "total": 30,
        "currency": "USD",
        "status": "shipped"
      },
      {
        "_links": {
          "self": {
            "href": "/orders/124"
          },
          "basket": {
            "href": "/baskets/97213"
          },
          "customer": {
            "href": "/customers/12369"
          }
        },
        "total": 20,
        "currency": "USD",
        "status": "processing"
      }
    ]
  },
  "currentlyProcessing": 14,
  "shippedToday": 20
}
```

### Link 对象

- href –string 必填项，它的内容可以是URL 或者URL模板。
- templated –bool 可选项，默认为false，如果href是URL模板则templated必须为true
- type –string 可选项，它用来表示资源类型
- deprecation –string 可选项，表示该对象会在未来废弃
- name –string 可选项， 可能当作第二个key，当需要选择拥有相同name的链接时
- profile –string 可选项，简要说明链接的内容
- title –string 可选项，用用户可读的语言来描述资源的主题
- hreflang –string 可选项，用来表明资源的语言 

### 建议 

- Self link 
- Link relations 链接关系



## API Demo

### 用户表


| id   | username   | nickname | sex  |
| ---- | ---------- | -------- | :--: |
| 1    | zuifeng    | 风哥     |  1   |
| 2    | feige      | 飞哥     |  1   |
| 3    | ouyangfeng | 锋哥     |  1   |
| 4    | tiantian   | 小甜甜   |  1   |

### 订单表

| id   | order_code   | Sku | amount  | user_id |
| ---- | :--------- | -------- | ---- | :--: |
| 1    | ord_001    | g_001     | 100    | 3 |
| 2    | ord_002      | g_002    | 98.5    | 2 |
| 3    | ord_003 | g_003  | 78.2    | 1 |
| 4    | ord_004   | g_004    | 33.5    | 3 |

### 货品表

| sku | spu   | spec |
| ---- | --------- | -------- |
| g_001    | sp_001    | 大    |
| g_002    | sp_001      | 巨大 |
| g_003    | sp_002 | 超级大 |
| g_004    | sp_002   | 很大  |



#### 获取用户信息

```shell
GET /users/1 
```

```json
{
	"username":"zuifeng",
	"nikename":"风哥"，
	"_links": {
	    "self": {
	    	"href":"/users/1"
	    },
      "update": {
         "href": "/users/1"
      },
     "orders": {
       "href": "/orders{?page,pageSize}",
       "templated": true
     }
	}
}
```

#### 获取订单

```shell
GET /orders/3
```

```json
{
	"order_code":"ord_003",
	"sku":"g_004"，
  "amount":"33.5",
	"_links": {
	    "self": {"href":"/orders/3"},
			"cancel": {"href": "/orders/3:cancel"},
	    "goods": {"href": "/goods/g_004"}
	}
}
```

#### 订单列表

```shell
GET /orders
```

```json
{
  "_links": {
    "self": {
      "href": "/orders"
    },
    "prev": {
      "href": "/orders?page=1"
    },
    "next": {
      "href": "/orders?page=3"
    },
    "find": {
      "href": "/orders{?id}",
      "templated": true
    }
  },
  "_embedded": {
    "orders": [
      {
        "order_code": "ord_003",
        "sku": "g_004",
        "amount": "33.5",
        "_links": {
          "self": {
            "href": "/orders/3"
          },
          "cancel": {
            "href": "/orders/3:cancel"
          },
          "goods": {
            "href": "/goods/g_004"
          }
        }
      },
      {
        "order_code": "ord_003",
        "sku": "g_004",
        "amount": "33.5",
        "_links": {
          "self": {
            "href": "/orders/3"
          },
          "cancel": {
            "href": "/orders/3:cancel"
          },
          "goods": {
            "href": "/goods/g_004"
          }
        }
      }
    ]
  },
  "currentPage": 2,
  "pageSize": 2
}
```







## 参考

https://zh.wikipedia.org/wiki/%E8%A1%A8%E7%8E%B0%E5%B1%82%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2

https://en.wikipedia.org/wiki/Hypertext_Application_Language

https://martinfowler.com/articles/richardsonMaturityModel.html#level0

https://datatracker.ietf.org/doc/html/draft-kelly-json-hal

