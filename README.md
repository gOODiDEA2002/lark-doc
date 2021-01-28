# Lark Version 1.5.0 说明

## 依赖组件

parent: spring boot 2.4.2

服务发现：nacos

数据库：mysql

消息队列：rocketmq

计划任务：xxl-job

服务间调用：spring mvc

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

* 模块类型
  
根据不同的应用场景，模块分为以下4种基本类型：

- **Api** 面向客户端的接口模块
- **Service** 对其它模块提供服务的模块
- **Msg** 消息处理模块
- **Task** 定时任务模块
   
> 建议: Api & Service 按前后端分别创建模块，如：
> front-api, admin-api
> front-service, admin-service

* 各类型模块占有端口
- **Api**: 1001 ~ 1999
- **Admin-Api**：2001 ~ 2999
- **Service**：3001 ~ 3999
- **Admin-Service**: 4001 ~ 4999
- **Msg-Handler**: 5001 ~ 5999
- **Task**: 6001 ~ 6999

### Api模块

Api模块分为 **契约** 和 **服务** 两种模块，分别是：

1. api-contract
   
1. api

> 参见: [API-开发说明文档](doc/api.md).

### Service模块

Service模块分为 **契约** 和 **服务** 两种模块，分别是：

1. service-contract

1. service

> 参见: [Service-开发说明文档](doc/service.md).

### Msg模块

Msg模块分为 **契约** 和 **处理器** 两种模块，分别是：

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
lark new project -group ${groupName} -artifact ${artifactName} ${projectDirname}
```

* 创建模块（进入项目目录后）

```
lark new ${moduleType} -group ${groupName} -artifact ${artifactName} ${moduleDirName}
```

> moduleType分为：
> - 前端接口：api
> - 前端接口协议：api-contract
> - 服务接口：service
> - 服务接口协议：service-contract
> - 消息发送及处理协议：msg-contract
> - 消息处理：msg-handler
> - 定时任务：task

> 参见: [lark-cli 使用说明文档](cli/README.md).



