# 数据库服务

**lark-db** 中封装了MySQL访问服务及组件。

## 场景

MySQL访问

## 快速上手

### POM

在 pom.xml 中添加如下依赖

```xml
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-db</artifactId>
</dependency>
```

### 配置

* 普通数据访问
  
在 application.yml 中添加如下配置

```yaml
lark:
  db:
    source:
      - user_master
      - user_slave
      - order_master_0
      - order_master_0
    user_master:
      name: techwis-demo
      address: db-00-qa.sanqlt.com:3306
      username: techwis
      password: wRcVxyxzJ@gKp4e6dGf.uzLu
      type: mysql
    user_slave:
      name: techwis-demo
      address: db-00-qa.sanqlt.com:3306
      username: techwis
      password: wRcVxyxzJ@gKp4e6dGf.uzLu
      type: mysql
    order_master_0:
      name: demo_order_0
      address: db-00-qa.sanqlt.com:3306
      username: techwis
      password: wRcVxyxzJ@gKp4e6dGf.uzLu
      type: mysql
    order_master_1:
      name: demo_order_1
      address: db-00-qa.sanqlt.com:3306
      username: techwis
      password: wRcVxyxzJ@gKp4e6dGf.uzLu
      type: mysql
```

* 分片数据访问(基于：ShardingSphere)

```yaml
lark:
  db:
    source:
      - user_master
      - user_slave
      - order_master_0
      - order_master_1
    #……在普通数据访问增加Shard配置
    shard:
      - order
    order:
      database: order_master_0, order_master_1
      route: order_master_$->{0..1}.t_order_$->{0..1}
      database-sharding:
        column: user_id
        algorithm: order_master_$->{user_id % 2}
      table-sharding:
        column: order_id
        algorithm: t_order_$->{order_id % 2}

```

> 分片目前只支持ID Hash方式
> 
### 应用

* 普通数据访问

 ```
    @Autowired
    DatabaseService databaseService;
    ……
    @Data
    @NoArgsConstructor
    @EqualsAndHashCode
    public class User {
        private int id;
        private int sex;
        private String name;
    }
    ……
    SqlQuery sqlQuery = databaseService.get( "user_master" );
    User user = sqlQuery.select("id", "name" )
                .from( "users" )
                .where( f( "id", 123 ) )
                .one( User.class );
    LOGGER.info( "DB User: >>> id: {}", user.getId() );
    ……
 ```

* 分片数据访问
 ```
    @Autowired
    DatabaseService databaseService;
    ……
    @Getter
    @Setter
    public class Order {
        private long orderId;
        private long userId;
        private int skuId;
    }
    ……
    SqlQuery sqlQuery = databaseService.getShard( "order" );
    Order order = sqlQuery.select( "order_id", "user_id", "sku_id" ).
        from( "order").
        where( f( "order_id", orderId )).
        one( Order.class );
    LOGGER.info( "DB Order: >>> id: {}", order.getUserId() );
    ……
 ```

## 常见问题

> JSD 语法参见：[JSD说明文档](jsd.md).

