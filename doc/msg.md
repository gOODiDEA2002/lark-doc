# 消息队列

**lark-msg** 中封装了消息队列组件，包含发送和订阅消息的一组接口，同时也提供了接口的一个默认 RocketMQ 消息队列实现。

> 考虑项目规范性，不建议单独使用消息发送和订阅功能，而是通过msg-contract和msg-handler两个类型的模块来提供消息发送和订阅功能

# 消息契约（msg-contract）

契约项目本着"谁处理，谁创建"的原则，由处理消息一方创建，发送消息一方负责调用

> 例如：会员模块需要在订单创建时增加用户积分，那么此契约应该由会员模块负责创建，订单模块负责引用此契约并发送消息。

> 注意: 预先在RocketMQ后台创建 **主题（Topic）** 和 **消费者（Consumer）**，消费者名称默认为Handler项目名，如：lark-example-msg-handler.

## 配置
  
在 pom.xml 中添加如下父依赖
```xml
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-msg-contract</artifactId>
    <version>1.5.0-SNAPSHOT</version>
    <relativePath/>
</parent>
```
## 定义

一个典型的消息契约目录结构如下：

```bash
├── topic #主题（Topic）
│   └── OrderTopic.java
├── message #消息实体（Message）
│   ├── OrderMessage.java
├── publisher #消息发送端 
│   ├── OrderPublisher.java
└── handler #消息订阅处理端 
└── └── OrderCreateHandler.java
```

* 主题（Topic）:OrderTopic.java
```java
public class OrderTopic {
    public static final String ORDER_CREATE_TOPIC = "ORDER_CREATE_TOPIC";
}
```
* 消息实体（Message）:OrderMessage.java
```java
@Data
public class OrderMessage {
    private long orderId;
    int userId;
    LocalDateTime createTime;
}
```
* 消息发送端:OrderPublisher.java
```java
public class OrderPublisher {

    private Publisher publisher;

    public OrderPublisher( Publisher publisher ) {
        this.publisher = publisher;
    }

    public void orderCreate(OrderMessage orderMessage ) {
        publisher.publish( OrderTopic.ORDER_CREATE_TOPIC, orderMessage );
        //延迟10分钟发送
        publisher.publish( OrderTopic.ORDER_CREATE_TOPIC, orderMessage, Duration.ofMinutes( 10 ));
    }
}
```
> 注意: 由于**RocketMQ**的限制，延迟的时间是个近似值，且最长不超过2小时。

* 消息订阅处理端:OrderCreateHandler.java
```java
@MsgHandler(topic = OrderTopic.ORDER_CREATE_TOPIC, threads = 2)
public abstract class OrderCreateHandler extends AbstractHandler<OrderMessage> {
}
```
## 发送消息

在 pom.xml 中添加对契约包依赖

```xml
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-example-msg-contract</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

在 application.yml 中添加如下配置

```yaml
lark:
  msg:
    address: mq-00-namesrv-qa.sanqlt.com:9876
    publisher: true
```

在代码中

```java
    @Autowired
    Publisher publisher;
    ……
    OrderMessage orderMessage = new OrderMessage();
    orderMessage.setOrderId( 123 );
    orderMessage.setUserId( 456 );
    orderMessage.setCreateTime(LocalDateTime.now());
    //
    OrderPublisher orderPublisher = new OrderPublisher( publisher );
    orderPublisher.orderCreate( orderMessage );
```

# 消息处理（msg-handler）

在 pom.xml 中添加如下父依赖

```xml
<parent>
    <groupId>lark</groupId>
    <artifactId>lark-starter-msg-handler</artifactId>
    <version>1.5.0-SNAPSHOT</version>
    <relativePath/>
</parent>
```
在 pom.xml 中添加对契约依赖
```xml
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-example-msg-contract</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

在 application.yml 中添加如下配置

```yaml
lark:
  msg:
    address: mq-00-namesrv-qa.sanqlt.com:9876
    subscriber: true
```

使用 **MsgApplication** 作为应用程序入口类。

```java
@SpringBootApplication
public class Bootstrap {
    public static void main(String[] args) {
        MsgApplication app = new MsgApplication(Bootstrap.class, args);
        app.run();
    }
}
```

**MsgApplication** 会自动扫描包内所有实现了 **Handler** 接口并带有 **MsgHandler** 注解的类，一个典型的消息处理项目目录结构如下：

一个典型的处理器如下：

```java
@Component
public class OrderCreateHandlerImpl extends OrderCreateHandler {
    Logger log = LoggerFactory.getLogger( OrderCreateHandlerImpl.class );
    
    @Override
    protected void process(OrderMessage msg, Message raw ) {
        log.info( ">>> order-:> orderId:{}, userId: {}, createTime:{}", msg.getOrderId(), msg.getUserId(), msg.getCreateTime() );
    }
}
```

## 消息使用建议

以下是开发消息相关应用时的一些常规性建议, 虽然不是强制要求, 但遵循这些建议往往会让你在实际开发中少走一些弯路

1. 消息是无序的
1. 关键流程或重要的消息处理应该考虑做幂等处理
1. 基于性能考虑, 大量日志型消息应该先尽量在内存进行适当合并(总消息体不要超过 1M 大小)再发送
1. 为避免影响正常业务, 日志消息的发送节点应该要跟正常业务消息进行隔离
1. 关键业务的消息处理器要尽量保证处理性能, 单个消息处理时间尽量控制在 60 秒以内
