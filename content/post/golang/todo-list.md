---
title: "任务清单小项目"
date: 2022-01-07T11:40:46+08:00
Description: ""
Tags: ["golang", "golang-project"]
Categories: ["golang"]
DisableComments: false
---

# 序


最近用`gin`+`gorm`写了一个小项目。所以有阵子没更新了。
项目十分简单，主要是以学习开发规范和熟悉代码编写为主，所以没有踩到什么大坑，这里就不多述了


![u=1952558653,2764759773&fm=26&fmt=auto](https://cdn.jsdelivr.net/gh/4927525/images@master/20220107/u=1952558653,2764759773&fm=26&fmt=auto.3hi8f5yajey0.webp)



# 任务清单-后端

**此项目使用`Gin`+`Gorm` ，基于`RESTful API`实现的一个任务清单**。

## 项目功能实现

- 登录鉴权（jwt-go）

- 清单 crud

- 分页

## 项目架构

```
TodoList/
├── api
├── conf
├── middleware
├── model
├── pkg
│  ├── util
├── routes
├── serializer
└── service
```

- api : 用于定义接口函数

- conf : 用于存储配置文件

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

- jwt-go

- mysql

## 项目接口

```bash
[GIN-debug] POST   /api/v1/user/register     --> TodoList/api.Register (3 handlers)
[GIN-debug] POST   /api/v1/user/login        --> TodoList/api.Login (3 handlers)
[GIN-debug] POST   /api/v1/task              --> TodoList/api.CreateTask (4 handlers)
[GIN-debug] GET    /api/v1/task/:id          --> TodoList/api.ShowTask (4 handlers)
[GIN-debug] GET    /api/v1/tasks             --> TodoList/api.ListTask (4 handlers)
[GIN-debug] PUT    /api/v1/task/:id          --> TodoList/api.UpdateTask (4 handlers)
[GIN-debug] DELETE /api/v1/task/:id          --> TodoList/api.DeleteTask (4 handlers)
```

## 项目运行

```bash
go mod tidy
```

```bash
go run main.go
```

## 感谢

[【Go语言实战】(3) Gin + Gorm 简单备忘录 | 含接口文档_面向生活编程-CSDN博客](https://blog.csdn.net/weixin_45304503/article/details/120680957)
