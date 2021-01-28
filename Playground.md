#Playground环境说明

##WIFI：Techwis-Ac68U（_5G）
密码：tian@A304
##依赖组件地址
#####DNS
192.168.99.92

#####Gitlab
http://192.168.99.90/
techwis/12345678

#####Nexus
http://192.168.99.90:8081/
admin/admin123

#####Harbor
http://192.168.99.91:8080/
gitlab-runner/Gitlab@123

#####Rancher
https://192.168.99.200:10443/
admin/rancher

#####Nacos
http://192.168.99.92:8848/nacos
nacos/nacos

###RocketMQ
192.168.99.92:9876
Console:http://192.168.99.92:8180/

#####XXL-job
http://192.168.99.92:8280/xxl-job-admin/
admin/123456

#####Redis
192.168.99.92:6379
12345678

#####MySQL
192.168.99.92:3306
lark/12345678

#####ES
http://192.168.99.92:9200/

#Gitlab-CI/CD
修改项目下的.gitlab-ci.yml，增加部署模块的信息，如：
-----
```
package_api:
  variables:
    MODULE_NAME: lark-example-api
  extends: .package
  script:
    - export project_dir=$(pwd)
    - cd /$project_dir/$MODULE_NAME
    - package.sh

deploy_api_playground:
  variables:
    MODULE_NAME: lark-example-api
    MODULE_TYPE: api
    INGRESS_HOST: api.foo.com
    INGRESS_PATH: /$MODULE_NAME
    CONTAINER_PORT: 1001
  extends: .deploy-playground
  script:
    - deploy.sh
```
-----

#访问
以下域名需连接WIFI（Techwis-Ac68U）或配置Host

#####HOST
192.168.99.201  traefik-console.foo.com
192.168.99.201	service.foo.com
192.168.99.201	api.foo.com
192.168.99.201	service.foo.com
192.168.99.201	task.foo.com
192.168.99.201	handler.foo.com

#####Traefik-Console
http://traefik-console.foo.com:8080

#####api:
http://api.foo.com{MODULE_INGRESS_PATH}/{MODULE_CONTROLLER_PATH}
http://api.foo.com/lark-example-api/test/hello.api

#####service：
http://service.foo.com{MODULE_INGRESS_PATH}/{MODULE_CONTROLLER_PATH}
http://service.foo.com/lark-example-service/test/hello.srv
……

