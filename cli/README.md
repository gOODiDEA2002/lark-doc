#lark-cli使用方式

根据操作系统下载lark并放到可执行目录下
mac/win/linux

#新建项目
```
lark new project -group ${groupName} -artifact ${artifactName} ${projectDirname}
```
#新建模块

进入项目所在目录
```
lark new ${moduleType} -group ${groupName} -artifact ${artifactName} ${moduleDirName}
```
#模块类型：moduleType
前端接口：api

前端接口协议：api-contract

服务接口：service

服务接口协议：service-contract

消息发送及处理协议：msg-contract

消息处理：msg-handler

定时任务：task

#各类型模块端口
Api: 1001 ~ 1999

Admin-Api：2001 ~ 2999

Service：3001 ~ 3999

Admin-Service: 4001 ~ 4999

Msg-Handler: 5001 ~ 5999

Task: 6001 ~ 6999

#示例:

#快递自提点服务平台(express-package-self-pickup-site)

#新建：项目
```
lark new project -group techwis. -artifact epsps-user saas-epsps-user 
```
#新建：Service & Contract
```
lark new service -group techwis. -artifact epsps-user-service service
```
```
lark new service-contract -group techwis. -artifact epsps-user-service-contract service-contract
```
```
lark new service -group techwis. -artifact epsps-user-admin-service admin-service
```
```
lark new service-contract -group techwis. -artifact epsps-user-admin-service-contract admin-service-contract
```
#新建：Api & Contract
```
lark new api -group techwis. -artifact epsps-user-api api
```
```
lark new api-contract -group techwis. -artifact epsps-user-api-contract api-contract
```
```
lark new api -group techwis. -artifact epsps-user-admin-api api
```
```
lark new api-contract -group techwis. -artifact epsps-user-admin-api-contract admin-api-contract
```
#新建：Task
```
lark new task -group techwis. -artifact epsps-user-task task
```
#新建：Msg Handler & Contract
```
lark new msg-handler -group techwis. -artifact epsps-user-msg-handler msg-handler
```
```
lark new msg-contract -group techwis. -artifact epsps-user-msg-contract msg-contract
```
#测试Service
```
curl --location --request POST 'http://127.0.0.1:3001/test/hello.srv' \
--header 'Content-Type: application/json' \
--data-raw '{
    "id": 123 
}'
```
#测试Api
```
curl -X POST "http://127.0.0.1:1001/test/hello.api" -d "id=123&name=xxx"
```