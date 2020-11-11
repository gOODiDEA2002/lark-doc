# Version 1.5.0 说明


#依赖组件
spring boot & spring cloud

服务发现：nacos

服务调用：OpenFeign

数据库：mysql & jdbc

缓存：redis & redisson

消息队列：rocketmq

计划任务：xxl-job

#项目模块类型
不同项目之间通过服务接口进行交互

前端接口协议：api-contract
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-api-contract</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
前端接口：api
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-api</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
服务接口协议：service-contract
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-service-contract</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
服务接口：service
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-service</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
消息发送及处理协议：msg-contract
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-msg-contract</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
消息处理：msg-handler
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-msg-handler</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
定时任务：task
```
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-task</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```
#脚手架工具: lark-cli
参见：cli/readme.md


