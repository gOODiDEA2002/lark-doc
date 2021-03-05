# 对象存储服务

lark.util.oss.OssService 封装了对Oss的常用方法

*注：目前只封装了minio

## 场景

保存文件

## 快速上手

### POM

在 pom.xml 中添加如下依赖

```xml
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-util-oss</artifactId>
</dependency>
```

### 配置

在 application.yml 中添加如下配置

```yaml
lark:
  util:
    oss:
      address: 192.168.99.97:9000
      username: minio
      password: 12345678
      type: minio
```

### 应用

 ```
    @Autowired
    OssService ossService;
    ……
    byte[] fileData = Files.readAllBytes(file.toPath());
    String objectName = ossService.upload( BUCKET_NAME, file.getName(), fileData );
    String objectUrl = ossService.getObjectUrl( BUCKET_NAME, objectName );
    fileData = ossService.download( BUCKET_NAME, objectName );
    ……
 ```

## 常见问题


