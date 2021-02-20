# Playground环境说明

**playground**环境用于熟悉框架和流程

## 前置条件
* WIFI 

SSID: techwis-ac68u / techwis-ac68u_5G 
  
> pwd：tian@A304

## 依赖组件地址

* Gitlab

http://code-repo.lark-cloud.com/

* Nexus

http://package-repo.lark-cloud.com/

> user: admin / pwd: admin123

> 参见: [私服配置文件](config/playground/nexus/settings.xml).  

* Harbor
  
http://image-repo-dev.lark-cloud.com

> user: devops / pwd: 0THJ83CRwEyYnuyS

* Nacos

register-dev.lark-cloud.com:8848

> user: nacos / pwd: nacos

* RocketMQ

mq-dev.lark-cloud.com:9876

> console: http://mq-console-dev.lark-cloud.com/

* XXL-job

http://schedule-dev.lark-cloud.com/xxl-job-admin/

> user: admin / pwd: 123456 / token: 12345678

* Redis

cache-dev.lark-cloud.com:6379

> pwd: 12345678

* MySQL

db-dev.lark-cloud.com:3306

> user: lark / pwd: 12345678

* ES

http://index-dev.lark-cloud.com:9200/

* k8s - Rancher 

https://192.168.99.200:10443/

> user: admin / pwd: rancher

## 应用访问地址

* Api

http://api.lark-cloud.com/${module-ingress-path}/${module-controller-path}

```bash
curl -X POST "http://api.lark-cloud.com/lark-example-api/test/hello.api" -d "id=123&name=xxx"
```

* Service

http://service.lark-cloud.com/${module-ingress-path}/${module-controller-path}

```bash
curl --location --request POST 'http://service.lark-cloud.com/lark-example-service/test/hello.srv' \
--header 'Content-Type: application/json' \
--data-raw '{
"id": 123
}'
```

* task

http://task.lark-cloud.com/${module-ingress-path}

```bash
url="http://task.lark-cloud.com/lark-example-task/run"
token="12345678"
task="TestTask"
echo -e "->request: $url $task\n"
curl --location --request POST $url \
--header 'Content-Type: application/json' \
--header 'XXL-JOB-ACCESS-TOKEN:$token' \
--data-raw '{
        "jobId": 1,
        "executorHandler": '\"$task\"',
        "executorParams": "",
        "logId": 1,
        "logDateTime":1586629003729
}'
```

* msg-handler

http://handler.lark-cloud.com/${module-ingress-path}

```bash
handler="OrderCreateHandlerImpl"
url="http://handler.lark-cloud.com/lark-example-msg-handler/run/${handler}"
echo -e "->request: $url $handler\n"
curl --location --request POST $url \
--header 'Content-Type: application/json' \
--data-raw '{
      "orderId":123,
      "userId":456
    }'
```