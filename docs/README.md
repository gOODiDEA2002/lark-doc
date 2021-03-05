---
home: true
footer: 京ICP备2021005006号
---
# Lark Version 1.5.0 说明

## 概述
**lark** 是一个基于Spring Boot的后端微服务框架。

为了标准化开发流程，根据不同场景，基于 **lark** 的项目通常把内部模块划分为以下几种基本类型：



- **api**
  
    面向前端/客户端的接口模块
  
- **api-contract** 
  
    面向前端/客户端的接口模块-接口定义

- **service** 
  
    提供服务的模块
  
- **service-contract** 
  
    提供服务的模块-接口定义  

- **msg-handler**
  
    消息处理模块
  
- **msg-contract**

    消息处理及发送-定义

- **task**
  
    定时任务执行模块

* 另外，为了更好的对前后台进行解耦，加入了以下针对后台的模块类型

- **admin-api**

  面向后台的接口模块

- **admin-api-contract**

  面向后台的接口模块-接口定义

- **admin-service**

  对后台调用提供服务的模块

- **admin-service-contract**

  对后台调用提供服务的模块-接口定义

一个典型的**lark**项目模块结构如下：

```bash
|-- file #文件模块
|   |-- api
|   |-- api-contract
|   |-- msg-contract
|   |-- msg-handler
|   |-- service
|   |-- service-contract
|   |-- task
|   |-- pom.xml
|-- user #会员模块
|   |-- admin-api
|   |-- admin-api-contract
|   |-- admin-service
|   |-- admin-service-contract
|   |-- api
|   |-- api-contract
|   |-- msg-contract
|   |-- msg-handler
|   |-- service
|   |-- service-contract
|   |-- task
|   |-- pom.xml
|-- merchant #商家模块
|-- goods #商品模块
|-- pay #支付模块
...
```

## 主要服务组件

spring boot 2.4.2

服务发现：nacos

服务调用：spring mvc

数据库：mysql

消息队列：rocketmq

定时任务：xxl-job

搜索：elasticsearch

## 组件使用说明

数据库：mysql
> 参考: [数据库-使用说明文档](doc/database.md).

缓存：redis & redisson
> 参考: [缓存-使用说明文档](doc/util.cache.md).

索引：elastic search
> 参考: [索引-使用说明文档](doc/util.index.md).

对象存储：minio
> 参考: [对象存储-使用说明文档](doc/util.oss.md).

## 模块开发说明

* 各类型模块占有端口
- **api**: 1001 ~ 1999
- **admin-api**：2001 ~ 2999
- **service**：3001 ~ 3999
- **admin-service**: 4001 ~ 4999
- **msg-handler**: 5001 ~ 5999
- **task**: 6001 ~ 6999

### Api模块

Api模块分为 **协议** 和 **服务** 两种模块，分别是：

1. api-contract
   
1. api

> 参见: [API-开发说明文档](doc/api.md).

### Service模块

Service模块分为 **协议** 和 **服务** 两种模块，分别是：

1. service-contract

1. service

> 参见: [Service-开发说明文档](doc/service.md).

### Msg模块

Msg模块分为 **协议** 和 **处理器** 两种模块，分别是：

1. msg-contract

1. msg-handler

> 参见: [Msg-开发说明文档](doc/msg.md).

### Task模块

task模块只有 **执行器** 模块，调度依赖于xxl-job：

> 参见: [Task-开发说明文档](doc/task.md).

## 脚手架工具: lark-cli

通过 **lark-cli** 可以创建项目及各类型模块

* 创建项目

```bash
lark new project -group groupName -artifact artifactName -port portSuffix projectDirname
```

* 创建模块（进入项目目录后）

```
lark new moduleType moduleDirName
```

> moduleType分为：
> - 前端接口：api
> - 前端接口协议：api-contract
> - 服务接口：service
> - 服务接口协议：service-contract
> - 消息发送及处理协议：msg-contract
> - 消息处理：msg-handler
> - 定时任务：task
> - 后台前端接口：admin-api
> - 后台前端接口协议：admin-api-contract
> - 后台服务接口：admin-service
> - 后台服务接口协议：admin-service-contract

> 参见: [lark-cli 使用说明文档](cli/README.md).



