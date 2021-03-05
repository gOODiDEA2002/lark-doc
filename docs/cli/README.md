#lark-cli使用方式

根据操作系统下载lark并放到可执行目录下
mac/win/linux

- 新建项目

```bash    
lark new project -group groupName -artifact artifactName -port portSuffix projectDirname
```
- 新建模块

```bash  
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

- 生成脚本示例
```bash
# new project
lark new project -group lark -artifact lark-demo -port 102 demo

cd lark
# new api
lark new api-contract api-contract
lark new api api

# new admin-api
lark new admin-api-contract admin-api-contract
lark new admin-api admin-api

# new service
lark new service-contract service-contract
lark new service service

# new admin-service
lark new admin-service-contract admin-service-contract
lark new admin-service admin-service

# new msg
lark new msg-contract msg-contract
lark new msg-handler  msg-handler

# new task
lark new task task
```
* 生成的模块名分别为：
- lark-demo-api
- lark-demo-api-contract
- lark-demo-admin-api
- lark-demo-admin-api-contract
- lark-demo-service
- lark-demo-service-contract
- lark-demo-admin-service
- lark-demo-admin-service-contract
- lark-demo-msg
- lark-demo-msg-contract
- lark-demo-task

* 模块占用端口分别为：
- lark-demo-api: 1102
- lark-demo-admin-api: 2102
- lark-demo-service: 3102
- lark-demo-admin-service: 4102
- lark-demo-msg: 5102
- lark-demo-task: 6102