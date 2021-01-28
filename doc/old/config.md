# 配置

配置是程序员永远的痛, 因为它的存在会大幅增加运维的复杂度. 虽然我们所有人都讨厌配置, 但在实际项目中我们又不得不使用配置. 没有配置, 我们无法处理大量环境依赖型的参数, 也无法动态插拔组件.

**lark** 同样未能免俗, 无法彻底消除各种应用程序配置, 但幸运的是 **lark** 针对配置从框架层面做了很多优化处理, 它支持全局配置+应用配置+profile切换+配置中心的多维度方案, 这也是全球顶级互联网巨头们使用的终极方案.

```mermaid
graph TD
    A(程序配置)
    E[环境变量]-->A
    P[PROFILE]-->A
    G[全局配置]-->A
    C[配置中心]-->A
    B[本地配置]-->A
```

## 环境变量

环境变量用于设置服务器相关的全局配置, 程序运行时会自动读取, 目前支持如下设置

名称        | 说明        | 示例值
---------- | ---------- | ----------
aiads_PROFILES_ACTIVE | 要激活的环境设置 | 如: aiads-stg
aiads_SERVER_IP | 服务器对外提供服务的 IP 地址 | 如: 192.168.0.228

## 配置文件

所有配置文件默认都使用 .conf 文件扩展名, 同时也兼容配置文件的实际扩展名(如配置文件 xxx.conf 是 xml 格式, 则也可以用 xxx.xml).

除了全局配置文件以外, 其它配置文件默认存放路径为程序打包后的 etc 子目录. 正常情况下程序在启动时会自动定位配置目录, 但你也可以在程序启动时用代码手动修改此目录:

```java
public static void main(String[] args) {
    ConfigManager.setConfigDir("/home/aiads/abc");
    // other code
}
```

> 注意: 配置由 **lark** 自动管理, 在合适的时机(比如第一次用到时)自动处理, 不需要开发者在程序中手动加载.

### global.conf

全局配置文件, 存放服务器级别的配置信息, 这些配置为此服务器上所有应用共用, 与某个具体的应用程序无关.

全局配置文件默认存放地址如下:

系统        | 目录
---------- | ----------
linux      | /home/aiads/etc
windows    | D:\etc
mac        | /etc/aiads

> 备注: 如果你不希望使用默认目录, 可以通过设置 **aiads_GLOBAL_CONFIG_PATH** 环境变量来修改.

* 示例

```xml
<settings>
    <setting key="etcd_nodes" value="http://192.168.51.102:2379" />
    <!--<setting key="etcd_nodes" value="http://192.168.50.56:12379,http://192.168.50.57:12379,http://192.168.50.52:12379" />-->
    <!-- etcd地址(new) -->
    <setting key="etcd.address" value="http://192.168.51.102:2379" />
    <!--<setting key="nsqlookupd_nodes" value="http://192.168.50.56:4161,http://192.168.50.57:4161" />-->
    <setting key="nsqlookupd_nodes" value="http://192.168.51.103:4161" />
    <!-- NSQ 发现节点地址 -->
    <setting key="nsq.subscribe.address" value="http://192.168.51.103:4161" />
    <setting key="default_service_ip" value="192.168.20.30" />
    <!-- 日志存储根目录 -->
    <setting key="log.path" value="/Users/noname/log" />
    <!-- 服务注册类型 -->
    <setting key="rpc.register.type" value="etcd" />
    <!-- 是否启用服务注册 -->
    <setting key="rpc.register.enabled" value="false" />
    <!-- 服务注册 IP -->
    <setting key="rpc.register.ip" value="192.168.20.30" />
</settings>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
etcd.address | etcd地址, 多个地址逗号分割 | http://192.168.51.102:2379
etcd_nodes   | 同 etcd.address, 历史遗留 |
nsq.subscribe.address | NSQ 订阅节点地址, 多个地址逗号分割 | http://192.168.51.103:4161
nsqlookupd_nodes | 同 nsq.subscribe.address, 历史遗留 |
log.path | 日志存储目录 | /Users/noname/log
rpc.discovery.enabled | 是否启用服务发现 | 默认 false
rpc.register.type | RPC 服务注册方式, 目前仅支持 etcd | etcd
rpc.register.enabled | 是否启用服务注册 | 默认 false
rpc.register.ip | 服务注册 IP | 192.168.51.151
default_service_ip | 同 rpc.register.ip, 历史遗留 |
rpc.client.timeout.read | RPC 客户端读超时时间, 单位毫秒 | 默认 30000
rpc.client.stats.type | RPC 客户端调用统计方式 | nsq/log, 默认 nsq
rpc.client.stats.enabled | 是否启用 RPC 客户端调用统计 | 默认 false
rpc.server.stats.type | RPC 服务端调用统计方式 | nsq/log, 默认 log
rpc.server.stats.enabled | 是否启用 RPC 服务端调用统计, 注意还需要在 log4j.conf 配置 `aiads.lark.net.rpc.server.$stats` 的 logger 才会起作用 | 默认 true

全局配置文件一般仅在 **lark** 框架内部使用, 如果要在业务项目中使用, 可以通过 *AppConfig* 类来获取相关配置:

```java
String ip = AppConfig.getDefault().getGlobal().getRpcRegisterIP();
// ...
```

### app.conf

应用程序配置文件, 存放预定义的应用程序基础配置和自定义的业务配置. 为了更好的管理和扩展, 应用程序配置文件是分章节的, 并且已经预设了 app/web/global/custom 四个章节.

* 示例

```xml
<config>
    <app>
        <add key="app_name" value="mx.common.sms.service"/>
        <add key="debug" value="true"/>
        <add key="global_env" value="mx"/>
        <add key="monitor_enabled" value="true"/>
        <add key="monitor_port" value="19513"/>
    </app>
    <web>
        <add key="http_addr" value=""/>
        <add key="http_port" value="10000"/>
        <!-- Form 验证是否启用 -->
        <add key="auth.form.enabled" value="true"/>
        <!-- Form 验证名称 -->
        <add key="auth.form.name" value="aiadsAuth"/>
        <!-- Form 验证 cookie 绑定的域名 -->
        <add key="auth.form.domain" value=""/>
        <!-- Form 验证安全凭据有效期，单位分钟-->
        <add key="auth.form.timeout" value="30"/>
        <!-- Form 验证安全凭据有效期是否支持滑动过期-->
        <add key="auth.form.slidingExpiration" value="true"/>
        <add key="access_log_enable" value="true"/>
    </web>
    <global>
        <add key="rpc.register.enabled" value="true" />
    </global>
    <custom>
        <add key="foo" value="bar"/>
    </custom>
</config>
```

* 预设章节说明

章节         | 说明
------------ | -------------
app | 程序基础设置
web | web 应用相关配置
global | 全局配置, 可以用此章节下相关设置来覆盖全局配置文件中的对应配置
custom | 自定义业务配置

* 配置说明

章节         | 设置           | 说明           | 示例值
------------ | ------------- | ------------- | ------------
app | app_name | 引用程序名称 | mx.common.sms.service
 | debug | 是否以调试模式运行 | 默认 false
 | global_env | 全局运行环境, 此配置会影响程序使用的 global.conf, 方便多环境下开发 | mx -> global.mx.conf
 | monitor_enabled | 是否启用 HTTP 监控 | 默认 true
 | monitor_port | HTTP 监控端口 | 19513
web | http_port | web 应用监控端口 | 8001
 | auth.form.enabled | 是否启用 form 认证 | 默认 false

应用程序配置可以通过 AppConfig 类来获取:

```java
// 获取 app 章节配置
String appName = AppConfig.getDefault().getAppName();
boolean monitorEnabled = AppConfig.getDefault().isMonitorEnabled();

// 获取 custom 章节配置
String foo = AppConfig.getDefault().getCustom().getString("foo", "");

// 获取 global 章节配置, 会自动与全局配置文件中的设置合并
String ip = AppConfig.getDefault().getGlobal().getRpcRegisterIP();
```

### cache.conf

缓存配置文件, 存放程序缓存相关设置, 包括缓存实现方式、过期时间等.

* 示例

```xml
<caches provider="memory" enabled="true" allowKeyMissing="true" defaultTime="30">
    <cache key="GetUser" time="10"/>
    <cache key="GetUserList" time="10" versionKey="GetUserListVersion"/>
</caches>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
provider | 缓存实现策略 | 有效值: redis/memory
enabled | 是否启用此缓存 | 默认值: true
allowKeyMissing | 是否允许未配置的键加入缓存 | 默认值: false
defaultTime | 默认缓存时间, 单位分钟 | 默认值: 30
cache[key] | 缓存键, 缓存系统中实际的缓存键会拼接上相关参数 | 如: GetUser
cache[time] | 缓存时间, 单位分钟, 如不配置则使用 defaultTime 的值 | 如: 10
cache[versionKey] | 缓存版本键, 仅在需要按版本批量移除缓存时才需要配置 | 如: GetUserListVersion

> 备注:
> 1. 当 provider 为 **redis** 时, 需要配置缓存用的 Redis 数据库(参见本文档 [db.redis.conf](config.md#h.1.11) 章节), 默认数据库名字为 **RedisCache**, 可以通过在 app.conf 配置文件中(或配置中心)的 custom 章节配置 **cache.redis.name** 来更改.
> 1. 名为 cache.remote.conf 的配置文件用于 lark-rpc 的客户端缓存, 自定义缓存对象时注意不要使用这个名字.

### log4j.conf

标准的 log4j 组件配置文件, XML 格式. **lark-util** 中实现了一个按日期分目录的日志记录器 `DailyFileAppender`, 推荐使用.

* 示例

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <!-- 输出到统一日志文件路径下 -->
    <appender name="aiadsFile" class="aiads.lark.util.log.DailyFileAppender">
        <!-- 设置File参数：日志输出文件名 -->
        <param name="File" value="aiads.log" />
        <!-- 设置日志存储目录, 如果不配置会从 global.conf 中加载 log.path 的设置 -->
        <!--<param name="Dir" value="/var/log/aiads" />-->
        <!--<param name="SplitByDateDir" value="false" />-->
        <!--<param name="MaxFileSize" value="100MB" />-->
        <!-- 设置输出文件项目和格式 -->
        <layout class="aiads.lark.util.log.CtxPatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p [%A] (%c:%L) - %m%n" />
        </layout>
    </appender>

    <!-- 输出到控制台中 -->
    <appender name="Console" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p: %m%n" />
        </layout>
    </appender>

    <!-- 按照名字空间进行日志输出控制 -->
    <logger name="aiads.lark.db.jsd.debug" additivity="false">
        <level value="DEBUG" />
        <appender-ref ref="Console" />
    </logger>

    <!-- 默认日志输出设置 -->
    <root>
        <priority value="INFO"/>
        <appender-ref ref="aiadsFile" />
        <appender-ref ref="Console"/>
    </root>
</log4j:configuration>
```

**lark** 框架如果在配置文件目录没有找到 log4j.conf(log4j.xml), 会启用默认设置日志配置, 默认配置大致如下所示:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="File" class="aiads.lark.util.log.DailyFileAppender">
        <param name="File" value="default.log" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p (%c:%L) - %m%n" />
        </layout>
    </appender>

    <appender name="Console" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p: %m%n" />
        </layout>
    </appender>

    <root>
        <priority value="DEBUG"/>
        <appender-ref ref="File" />
        <appender-ref ref="Console"/>
    </root>
</log4j:configuration>
```

> 注意: 只有使用 **LoggerManager** 类获取的 **Logger** 对象才会使用此配置, 如果直接使用 log4j 或 slf4j 组件则可能不会生效(取决于 **lark** 框架内部组件使用日志相关类的时机). 鉴于此, 在使用 **lark** 时, 请总是使用 **lark-util** 中的 **LoggerManager** 类来获取日志记录对象.

### remote.client.conf

RPC 客户端配置文件, 此配置文件可以省掉, 如果不存在此配置文件, 默认会走配置中心去获取 RPC 服务节点.

* 示例

```xml
<servers>
    <server name="ecommerce" type="simple" group="" address="192.168.50.52:15011" discovery="true" description="后产品服务">
        <setting name="MaxConnections" value="100"/>
    </server>
    <server name="number" type="simple" group="" address="192.168.50.22:9625" discovery="false" description="发号器服务">
        <setting name="MaxConnections" value="100"/>
    </server>
</servers>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
name | 服务程序名称 | mx.common.sms.service
alias | 服务程序别名, 如果指定, 则会用此名称去进行服务发现 |
type | 服务类型, 目前仅支持 simple |
group | 服务分组名称, 指定分组后使用服务发现方式时将只能连接到此分组的服务, 分组可以在配置管理后台进行配置 | front
version | 指定要连接的服务版本, 如果指定, 则只会请求此版本的服务端 | 1.0.1
address | 服务地址, 当 discovery 为 false 时会使用此地址进行直连 | 192.168.50.52:15011
discovery | 是否启用服务发现, 如果不指定, 会使用 global.conf 中的 rpc.discovery.enabled 配置 |
setting | 服务参数, 目前支持如下设置 |
setting[Cacheable] | 是否启用客户端缓存 | 默认 false
setting[LoadBalancer] | 负载均衡方式 | RR: 轮盘, R: 随机, 默认为随机方式
setting[MaxConnections] | 连接池最大连接数 | 默认 500
setting[ConnectTimeout] | 连接超时时间, 单位毫秒 | 默认 10 * 1000(10 秒)
setting[ReadTimeout] | 读取超时时间, 单位毫秒 | 默认 30 * 1000(30 秒)
setting[WriteTimeout] | 写入超时时间, 单位毫秒 | 默认 30 * 1000(30 秒)

### remote.server.conf

RPC 服务端配置文件, 如果在构造 `RpcServer` 手动传入 `ServerOptions` 参数, 则此配置文件也可以省掉.

* 示例

```xml
<servers>
    <server name="goods" type="simple" register="true" address=":9011" description="后产品商品服务">
        <setting name="MaxConnections" value="1000"/>
    </server>
</servers>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
name | 服务程序名称 | mx.common.sms.service
type | 服务类型, 目前仅支持 simple |
address | 服务绑定和注册地址, 建议这里仅指定端口, 不指定 IP | :15011
register | 是否启用服务注册, 如果不指定, 会使用 global.conf 中的 rpc.register.enabled 配置 | 默认 false
version | 服务版本 | 1.0.1
setting | 服务参数, 目前支持如下设置 |
setting[ServicePackage] | 可选, 要额外扫描服务接口的包(仅会扫描其中的接口类), 多个用逗号分隔 | 如: aiads.abc.service
setting[ServiceClass] | 可选, 要额外注册的服务类(接口类或实现类), 多个用逗号分隔 | 如: aiads.abc.service.TestService
setting[MaxClients] | 最大允许的客户端连接数 | 默认 1000
setting[ConnectTimeout] | 连接超时时间, 单位毫秒 | 默认 10 * 1000(10 秒)
setting[ReadTimeout] | 读取超时时间, 单位毫秒 | 默认 30 * 1000(30 秒)
setting[WriteTimeout] | 写入超时时间, 单位毫秒 | 默认 30 * 1000(30 秒)
setting[MethodNameIgnoreCase] | 服务方法名不区分大小写, 此选项主要用来兼容旧版框架使用 soa-spring 方式调用的客户端, 一般无需开启 | 默认 false

### urlserver.conf

站点配置文件, 提供获取网站地址的统一入口.

* 示例

```xml
<?xml version="1.0" encoding="utf-8" ?>
<servers>
    <server name="www" url="http://www.aiads.com"/>
    <server name="movie" url="http://movie.aiads.com"/>
    <server name="people" url="http://people.aiads.com"/>
    <server name="video" url="http://video.aiads.com"/>
</servers>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
name | 站点名称, 必须唯一 | movie
url | 站点地址 | [http://movie.aiads.com](http://movie.aiads.com)

### url.conf

站点页面地址配置文件, 一般情况, 此配置文件中只配置了不包含站点域名的相对地址, 所以它需要依赖 *urlserver.conf* 配置文件.

* 示例

```xml
<?xml version="1.0"?>
<urls version="2.0">
    <server name="www">
        <url name="DisneyDVD" path="/Disney/DVD/{0}"/>
        <url name="SonyDVD" path="/Sony/DVD/{0}"/>
        <url name="ShowTimeMovie" path="/showtime/{0}/"/>
    </server>
    <server>
        <!--IMDB电影URL-->
        <url name="IMDBMovie" path="http://www.imdb.com/title/tt{0}/"/>
    </server>
</urls>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
name(server) | 站点名称 | movie
name(url) | 页面名称, 必须唯一 | movie.index
path | 页面地址, 如果 server 节点设置了 name, 则为相对地址, 否则为绝对地址 | /showtime/{0}/

**lark-util** 中的 `UrlConfig` 类封装了获取页面地址的相关方法

```java
String url = UrlConfig.getUrl("ShowTimeMovie", 38260);
```

> 注意: 在需要使用 URL 的地方都应该从配置文件加载, 而不能直接在页面中写死, 特别是跨站点的地址.

### db.sql.conf

SQL 数据库配置文件

* 示例

```xml
<databases>
    <database name="aiads" provider="mssql" address="jdbc:sqlserver://192.168.50.104:1433;databaseName=aiads">
        <setting name="Username" value="aiadsuser"/>
        <setting name="Password" value="aiadsuser0301"/>
    </database>
    <database name="aiadsgroup" provider="mysql" address="jdbc:mysql://192.168.50.41:3306/test" >
        <setting name="Username" value="aiadsuser"/>
        <setting name="Password" value="aiadsuser123"/>
        <setting name="MaxIdleConns" value="1"/>
        <setting name="MaxOpenConns" value="100"/>
    </database>
</databases>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
database[name] | 数据库名称 | 如: ECommerce
database[provider] | 数据库类型 | 目前支持 mysql/mssql
database[driver] | 数据库驱动类型, 一般无需设置 | 如: com.mysql.jdbc.Driver
database[address] | 数据库连接字符串 | 如: jdbc:mysql://192.168.50.41:3306/test
setting | 数据库连接参数, 目前支持如下设置 |
setting[Username] | 用户名 |
setting[Password] | 密码 |
setting[MaxIdleConns] | 最大空闲连接数 | 默认值 10
setting[MaxOpenConns] | 最大连接数 | 默认值 100
setting[AcquireTimeout] | 获取连接超时时间, 单位秒 | 默认值 10 秒

### db.mongo.conf

MongoDB 数据库配置文件

* 示例

```xml
<databases>
    <database name="ECommerce">
        <setting name="ConnString" value="192.168.50.24:27017/ECommerce"/>
    </database>
</databases>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
database[name] | 数据库名称 | 如: ECommerce
setting | 数据库连接参数, 目前支持如下设置 |
setting[ConnString] | 数据库连接字符串 | 如: 192.168.50.24:27017/ECommerce
setting[ConnectTimeout] | 连接数据库超时时间(毫秒) | 默认值 10000 (10秒)
setting[SocketTimeout] | 数据库读写超时时间(毫秒) | 默认值无(永不超时)
setting[MaxPoolSize] | 每主机最大连接数 | 默认值 100
setting[MinPoolSize] | 每主机最小连接数 | 默认值 0
setting[ReadPreference] | 副本集读节点选择策略 | 默认值 primary, 有效值: primary, primaryPreferred, secondary, secondaryPreferred, nearest

### db.redis.conf

Redis 数据库配置文件, 使用 `RedisClient` 或 `RedisCacheService` 时会依赖此配置文件. 此配置文件可以省掉, 如果不存在此配置文件, 默认会走配置中心去获取数据库配置.

* 示例

```xml
<databases>
    <database name="RedisCache">
        <servers>
            <add host="192.168.50.41" port="6379" />
            <add host="192.168.50.42" port="6379" />
        </servers>
        <settings>
            <!-- <add name="Type" value="sentinel"/> -->
            <!-- <add name="MasterNames" value="wr-5329,wr-5339"/> -->
            <!-- <add name="Password" value=""/> -->
            <add name="MinPoolSize" value="1"/>
            <add name="MaxPoolSize" value="100"/>
            <add name="ConnectTimeout" value="5000"/>
            <add name="ReadTimeout" value="10000"/>
        </settings>
    </database>
</databases>
```

* 配置说明

设置         | 说明           | 示例值
------------ | ------------- | ------------
name | 数据库名称 | RedisCache
servers | 分片节点列表, **lark** 框架会采用一致性哈希来获取每次的请求节点 |
servers[host] | 节点地址, 可以为 IP 地址或域名 | 如: 192.168.50.41
servers[port] | 节点端口 | 如: 6379
setting | 数据库连接参数, 目前支持如下设置 |
setting[Type] | Redis 部署模式，有效值: sentinel, ms-sentinel | 默认值无, 表示主从模式
setting[MasterNames] | sentinel 模式下的 Master Name, 多个名称用逗号隔开 |
setting[Password] | Redis 服务器密码 |
setting[MinPoolSize] | 连接池中最少保持的连接数 | 默认值 1
setting[MaxPoolSize] | 连接池中最大允许的连接数 | 默认值 100
setting[ConnectTimeout] | 连接超时时间, 单位毫秒 | 默认 5 * 1000(5 秒)
setting[ReadTimeout] | 读写超时时间, 单位毫秒 | 默认 5 * 1000(5 秒)

### spring.conf

Spring 框架配置文件, 由于 Spring 在 Java 世界的统治地位短时间不可动摇, **lark** 对它提供了内置支持以简化大家的配置, 当存在 spring.conf 或 spring.xml 文件时, 所有从 Application 派生的应用程序启动类都会自动加载此配置文件.

## Profile

程序开发完成后, 会在不同的环境下进行部署, 如开发/测试/验收/生产环境等. 不同环境的配置很可能是不一样的, 如数据库地址等, 为了使配置能够适配多套部署环境, **lark** 引入了 Profile 设置文件来进行预处理, Profile 设置文件是一系列名为 `app[-<profile>].properties` 的设置文件, **lark** 框架在加载配置文件时, 会自动用当前激活的 Profile 配置文件中的设置做替换.

### Profile 命名

名字没有限制, 但推荐使用下面的命名方式:

环境         | 命名           | 示例
------------ | ------------- | ------------
开发环境 | `<xxx>-dev` | aiads-dev
测试环境 | `<xxx>-qas` | aiads-qas
仿真环境 | `<xxx>-stg` | aiads-stg
生产环境 | `<xxx>-prd` | aiads-prd

**lark** 框架对符合上述命名规范的 Profile 做了内置的优化与适配处理.

### 默认 Profile

为便于开发，从 _1.0.33_ 版本起，名为 **default** 的 Profile 被 **lark** 作为保留的默认 Profile，当没有任何 Profile 激活，且存在 **app-default.properties** 配置文件时，就会自动激活默认环境设置。

### 激活 Profile

有两种方式可以激活 Profile

a) **jctl** 启动程序时传递 -p 参数

```bash
jctl start <app_path> -p aiads-dev,local
```

b) 配置 **aiads_PROFILES_ACTIVE** 系统环境变量

```bash
export aiads_PROFILES_ACTIVE=aiads-stg
```

你可以同时激活多个 Profile, 用逗号隔开即可

```bash
jctl start <app_path> -p aiads-dev,local
```

> 注意: `app[-<profile>].properties` 同时也替换了 Spring Boot 默认的 `application.properties`, 所以你应该在 app.properties 配置文件中去设置原 Spring Boot 的配置.

## 配置中心

配置中心是一个集中式配置管理平台, 它主要由配置管理后台和配置服务两部分组成, 配置中心天生是运行环境隔离的.

### 配置管理后台

配置中心的所有配置都是通过配置管理后台来维护的, 大致包含如下几个模块.

### 配置服务

配置服务提供读取各种配置的接口, 它跟 RPC 服务类似, 启动后也会注册到服务注册中心, 但为了方便各个不同平台能非常简单的接入, 它采用了 HTTP 协议而不是自定义的 TCP 协议.

**lark** 框架中已经完整封装了配置服务的几乎所有接口, 所以正常情况下你不需要手动去访问配置中心, 应用程序会在需要的时候自动去配置中心加载配置. 比如你在项目中用到了 `sms` 这个数据库(假设使用 jsd 作为数据访问组件), 如果 **jsd** 未能在本地的 **db.sql.conf** 中找到对应的配置, **lark** 的配置组件就会去配置中心查找此数据库的配置信息.

## 配置使用建议

理想情况下, 程序在构建完成并打包后应该是一个致密的盒子, 不能有外在因素去修改它, 这样发布上线流程才能保持极度的简洁, 不需要任何运维人员的干预, 同时后续做容器化改造也很方便.

要做到这些, 需要遵循以下原则:

1. 与运行环境无关的基础配置放在配置文件中, 如: 程序名称, 服务端口, Spring AOP配置等
1. 与运行环境相关的动态配置由配置中心统一管理, 如: 操作员帐号, 数据库分片等
1. 与安全相关的配置必须由配置中心统一管理, 如: 数据库, 支付密钥等

整理后, 一个典型的 RPC 服务程序配置目录仅仅只需要包含 app.conf 和 cache.conf 即可:

```bash
etc
├── app.conf
└── cache.conf
```