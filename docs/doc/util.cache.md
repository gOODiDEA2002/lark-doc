# 缓存服务

lark.util.cache.CacheService 封装了常用缓存的方法
lark.util.cache.LockService 封装了分布式锁的方法
lark.util.cache.RateLimitService 封装了限流的方法

## 场景


## 快速上手

### POM

在 pom.xml 中添加如下依赖

```xml
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-util-cache</artifactId>
</dependency>
```

### 配置

在 application.yml 中添加如下配置

```yaml
lark:
  util:
    cache:
      address: 192.168.99.92:6379
      password: 12345678
```
### 缓存应用

 ```java
    @Autowired
    CacheService cacheService;
    ……
    UserItem item = new UserItem();
    item.setId(123);
    item.setName("123");
    cacheService.set( "testuser", item, Duration.ofMinutes( 3 ) );
    item = cacheService.get( "testuser", UserItem.class );
    ……
 ```

### 分布式锁应用

 ```java
    @Autowired
    LockService lockService;
    ……
    int orderId = 1000;
    String lockKey = String.format( "test-api-order-%d", orderId );
    //等待锁，180秒
    int waitSec = 180;
    //获取锁60秒以后自动解锁
    int autoUnlock = 60;
    int result = lockService.tryLock( lockKey, waitSec, autoUnlock, () -> {
        LOGGER.info( "Thread:{} lockService get lock", Thread.currentThread().getName() );
        //耗时操作，如：
        try {
            Thread.sleep(1000 );
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LOGGER.info( "Thread:{} lockService unlock", Thread.currentThread().getName() );
        //返回值可以为多种类型，这里是整型
        return 0;
    });
 ```

### 限流应用

 ```java
    @Autowired
    RateLimitService rateLimitService;
    ……
    String rateLimitKey = String.format( "test-api-order-%d", orderId );
    //60秒执行1次
    int rate = 1;
    int intervalSecond = 60;
    boolean exec = rateLimitService.tryProcess( rateLimitKey, rate, intervalSecond, () -> {
        LOGGER.info( "Thread:{} rateLimitService exec {}/{}s", Thread.currentThread().getName(), rate, intervalSecond );
    });
    if ( !exec ) {
        LOGGER.info( "Thread:{} rateLimitService Limited {}/{}s", Thread.currentThread().getName(), rate, intervalSecond );
    }
 ```

## 常见问题