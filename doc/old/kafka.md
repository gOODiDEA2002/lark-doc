# Kafka

Kafka 是一个分布式发布-订阅消息传递系统。

## 主要特性

* 高吞吐量、低延迟：Kafka 每秒可以处理几十万条消息，它的延迟最低只有几毫秒
* 可扩展性：Kafka 集群支持热扩展
* 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失
* 容错性：允许集群中节点失败（若副本数量为 n, 则允许 n-1 个节点失败）
* 高并发：支持数千个客户端同时读写
* 支持实时在线处理和离线处理：可以使用 Storm 这种实时流处理系统对消息进行实时进行处理，同时还可以使用 Hadoop 这种批处理系统进行离线处理

## 使用场景

* 日志收集：一个公司可以用 Kafka 可以收集各种服务的日志，通过 Kafka 以统一接口服务的方式开放给各种 consumer，例如 Hadoop、Hbase、Solr 等。
* 消息系统：解耦和生产者和消费者、缓存消息等。
* 用户活动跟踪：Kafka 经常被用来记录 Web 用户或者 App 用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到 Kafka 的 topic 中，然后订阅者通过订阅这些 topic 来做实时的监控分析，或者装载到 Hadoop、数据仓库中做离线分析和挖掘。
* 运营指标：Kafka也经常用来记录运营监控数据，包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
* 流式处理：比如 Spark Streaming 和 Storm。

## 快速上手

Spring Boot 2.0 对 Kafka 提供了完整的集成支持。

### 创建项目

在 pom.xml 中添加如下依赖

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

### 配置

```yaml
spring:
  kafka:
    consumer:
      group-id: test-app
      auto-offset-reset: earliest
      key-deserializer:  org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.JsonDeserializer
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.JsonSerializer
    bootstrap-servers: localhost:9092
    listener:
      concurrency: 3
```

### 序列化器

采用自定义的 JSON 序列化器，以方便跨平台通讯。

### 发送消息

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class Sender {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public void send(String message){
        kafkaTemplate.send("test", message);
    }
}
```

### 订阅消息

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Service;

@Service
public class Receiver {
    @KafkaListener(topics = "test", gour)
    public void listen(@Payload String message, @Headers MessageHeaders headers) {
    }
}
```

## 常见问题

### Kafka如何保证数据的不丢失

#### 生产者数据的不丢失

新版本的 producer 采用异步发送机制。KafkaProducer.send(ProducerRecord) 方法仅仅是把这条消息放入一个缓存中(即 RecordAccumulator，本质上使用了队列来缓存记录)，同时后台的 IO 线程会不断扫描该缓存区，将满足条件的消息封装到某个 batch 中然后发送出去。显然，这个过程中就有一个数据丢失的窗口：若 IO 线程发送之前 client 端挂掉了，累积在 accumulator 中的数据的确有可能会丢失。 Kafka 的 ack 机制：在 Kafka 发送数据的时候，每次发送消息都会有一个确认反馈机制，确保消息正常的能够被收到。

* 如果是同步模式：ack 机制能够保证数据的不丢失，如果 ack 设置为 0，风险很大，一般不建议设置为 0
```properties
    producer.type=sync 
    request.required.acks=1
```
* 如果是异步模式：通过 buffer 来进行控制数据的发送，有两个值来进行控制，时间阈值与消息的数量阈值，如果 buffer 满了数据还没有发送出去，如果设置的是立即清理模式，风险很大，一定要设置为阻塞模式
```properties
    producer.type=async 
    request.required.acks=1 
    queue.buffering.max.ms=5000 
    queue.buffering.max.messages=10000 
    queue.enqueue.timeout.ms = -1 
    batch.num.messages=200
```
> 结论：producer 有丢数据的可能，但是可以通过配置保证消息的不丢失

#### 消费者数据的不丢失

* 如果在消息处理完成前就提交了 offset，那么就有可能造成数据的丢失。由于 Kafka consumer 默认是自动提交位移的，所以在后台提交位移前一定要保证消息被正常处理了，因此不建议采用很重的处理逻辑，如果处理耗时很长，则建议把逻辑放到另一个线程中去做。为了避免数据丢失，现给出两点建议：
```properties
    # 关闭自动提交位移，在消息被完整处理之后再手动提交位移
    enable.auto.commit=false
```
* 如果使用了 Storm，要开启 Storm 的 ackfail 机制；
* 如果没有使用 Storm，确认数据被完成处理之后，再更新 offset 值。低级 API 中需要手动控制 offset 值。通过 offset commit 来保证数据的不丢失，kafka 自己记录了每次消费的 offset 数值，下次继续消费的时候，接着上次的 offset 进行消费即可。