---
title: "基于 WebSocket + MongoDB 的IM即时聊天Demo"
date: 2022-01-11T10:27:37+08:00
Description: ""
Tags: ["golang", "websocket"]
Categories: ["golang"]
DisableComments: false
---



# 序

又过了几天，上个小项目还没来得及消化，就稀里糊涂的过了个周末，昨天收了点尾，现在刚好有时间安排！

<img title="点击查看源网页" src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201804%2F24%2F20180424234945_FA4QP.thumb.700_0.jpeg&refer=http%3A%2F%2Fb-ssl.duitang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1644460567&t=7e4832ec5de2ef91ba66bd2c2cf33d26" alt="" data-align="inline">



# 写在前面

> 这个项目是基于WebSocket + MongoDB + MySQL + Redis。  
> 业务逻辑很简单，只是两人的聊天。



- `MySQL` 用来存储用户基本信息
- `MongoDB` 用来存放用户聊天信息
- `Redis` 用来存储处理过期信息





## 项目功能实现

- 登录鉴权（jwt-go）

- 清单 crud

- 分页

## 项目架构

```
TodoList/

├── api

├── conf

├── e

├── middleware

├── model

├── pkg

│  ├── util

├── routes

├── serializer

└── service
```

- api : 用于定义接口函数

- conf : 用于存储配置文件
- e: 错误代码文件

- middleware : 应用中间件

- model : 应用数据库模型

- pkg / util : 工具函数

- routers : 路由逻辑处理

- serializer : 将数据序列化为 json 的函数

- service : 接口函数的实现

## 项目依赖

- go1.16

- gin

- gorm

- mysql
- redis
- mongodb

## 项目接口

```bash
[GIN-debug] GET    /ping                     --> chat/routers.NewRouters.func1 (5 handlers)
[GIN-debug] POST   /user/register            --> chat/api.UserRegister (5 handlers)
[GIN-debug] GET    /ws                       --> chat/service.Handler (5 handlers)
```

## 项目运行

```bash
go mod tidy
```

```bash
go run main.go
```



# 地址

[4927525/go-chat · GitHub](https://github.com/4927525/go-chat)[4927525/go-chat · GitHub](https://github.com/4927525/go-chat)

# 感谢

[【Go语言实战】(7) 基于 WebSocket + MongoDB 的IM即时聊天Demo_面向生活编程-CSDN博客_golang 即时通讯](https://blog.csdn.net/weixin_45304503/article/details/121787022)
