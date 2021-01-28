# 缓存

缓存是提升程序性能最有效的手段之一, 根据实现方式的不同, 大致可以分成如下类型

* 进程内内存缓存
* 进程外内存缓存
* 文件缓存
* 数据库缓存

文件缓存和数据库缓存在前些年还比较常见, 但在进入内存白菜价的年代后已经逐渐式微, 如今仅在某些特定场景中才会使用, 所以 **lark** 中目前并未提供这两种缓存策略的实现.

进程内内存缓存和进程外内存缓存目前是最常用的, **lark** 中已经直接集成了, 其中后者是基于 Redis 实现的.

## 基本准则

最基本的暂时只列出一条

1. 所有面向外部用户的数据库查询请求必须添加缓存
1. 所有面向外部用户的数据库查询请求必须添加缓存
1. 所有面向外部用户的数据库查询请求必须添加缓存

但重要的事情要说三遍~

## 缓存配置

请参考[配置解析](config.md#h.2.3)中缓存配置相关章节.

## 使用方法

**lark** 中的缓存组件使用了策略模式, 通过缓存配置文件中的设置来注入对应的缓存实现, 使用时只需要操作 `Cache` 类即可, 而不需要直接跟缓存实现类打交道.

### 设置缓存

`Cache` 类提供了静态方法 `set` 来设置缓存. 根据缓存策略的不同, 对缓存的对象可能有序列化要求, 比如 redis 缓存实现需要对象支持 JSON 序列化和反序列化.

```java
// 缓存字符串
Cache.set("bar", "foo", "123");

// 缓存对象
lark.util.index.user.UserEntityy.User userEntity = new lark.util.index.user.UserEntityy.User(1);
Cache.set(userEntity, "GetUser", userEntity.getID());
```

> 注意: 因为 Protocol Buffer 插件直接生成的 Builder 模式的 DTO 对象并不支持 JSON 序列化, 所以不要去缓存这种对象.

### 获取缓存

不同于 `set` 只有一个方法, 获取缓存的 `get` 方法有一系列重载

```java
String value = Cache.get(String.class, "foo", "123");
System.out.println("value: " + value);
// print: bar

lark.util.index.user.UserEntityy.User userEntity = Cache.get(lark.util.index.user.UserEntityy.User.class, "GetUser", 1);
```

还可以指定没有获取到缓存时的默认值

```java
// 如果缓存不存在, 则返回 abc
String value = Cache.get(String.class, () -> "abc", "foo", "123");
```

由于 Java 的泛型是伪泛型, 所以获取泛型对象时会比较麻烦

```java
Goods goods = new Goods();
goods.setId(123);
goods.setName("test");
List<Goods> list = new ArrayList<>();
list.add(goods);

Cache.set(list, "GetGoodsList");
list = Cache.get(new TypeWrapper<List<Goods>>() {
}, "GetGoodsList");
System.out.println(list.get(0).getName());
// print: test
```

基于性能和便捷性考虑, 这种情况可能增加一个辅助集合实体更好些

```java
@Getter
@Setter
public class GoodsList {
    private List<Goods> items;
}
```

### 移除缓存

移除单个缓存

```java
Cache.remove("foo", "123");
```

如果缓存键配置了版本键, 则可以按版本批量移除缓存

```java
lark.util.index.user.UserEntityy.User user1 = new lark.util.index.user.UserEntityy.User(1);
lark.util.index.user.UserEntityy.User user2 = new lark.util.index.user.UserEntityy.User(2);
Cache.set(user1, "GetUser", 1);
Cache.set(user2, "GetUser", 2);
// 移除了所有 GetUser 键的缓存
Cache.removeByVersion("GetUserVersion");
```

> 注意: 启用按版本批量移除有一定的性能损失(多一次版本检查), 故只有在必须使用时才开启.

## 多缓存策略共存

一般情况下一个应用使用一种缓存策略就足够了, 但某些特殊场合会用到多种, 比如用户帐户的基础信息可能极少改变, 即使有多个程序实例, 用进程内内存缓存也能基本满足需求, 同时性能也更好. 在 **lark** 中可以用添加多个缓存配置文件的方式来启用多缓存策略, 比如默认缓存文件 cache.conf 配置为

```xml
<caches provider="redis" enabled="true" allowKeyMissing="false" defaultTime="30">
    <cache key="GetGoods" time="10"/>
    <cache key="GetGoodsList" time="10" versionKey="GetGoodsListVersion"/>
</caches>
```

我们再添加一个缓存配置文件 cache.userEntity.conf

```xml
<caches provider="memory" enabled="true" allowKeyMissing="true" defaultTime="30">
    <cache key="GetUser" time="10"/>
    <cache key="GetUserList" time="10" versionKey="GetUserListVersion"/>
</caches>
```

就可以在代码中用如下方式来获取和使用第二种缓存策略了

```java
CacheWrapper wrapper = Cache.getWrapper("userEntity");

wrapper.set("noname", "GetUser", 123);
String userEntity = wrapper.get(String.class, "GetUser", 123);
System.out.println("value: " + value);
```

## 高级设置

针对具体的缓存策略实现, 还可以在 app.conf 中(或者配置中心)开启一些高级设置, 比如数据压缩. 目前已经支持的设置如下

```xml
<custom>
    <add key="cache.redis.name" value="RedisCache"/>
    <add key="cache.redis.compress.enabled" value="true"/>
    <add key="cache.redis.compress.threshhold" value="32768"/>
</custom>
```

设置         | 说明           | 示例值
------------ | ------------- | ------------
cache.redis.name | Redis 缓存数据库名称 | 默认值: RedisCache
cache.redis.compress.enabled | Redis 缓存是否启用压缩 | 默认值: false
cache.redis.compress.threshhold | Redis 缓存压缩阈值 | 默认值: 32768(32K)


> 备注: 一般情况下, 启用压缩会导致性能下降, 所以除非你的缓存对象真的非常大, 否则不建议开启.