# jsd 快速上手指南

**jsd** 是一个简洁、优雅、高性能数据访问框架, 它使用流式类 SQL 查询接口, 主要优点如下

* 接口简单, 上手快
* 极高的性能, jsd 能根据数据库类型自动生成高效的 SQL 语句(无限接近 jdbc 的性能)
* 兼容 MySQL / SQL Server 等多种数据库, 无缝切换
* 全部采用参数化查询, 杜绝 SQL 注入漏洞

## 准备工作

为了演示，我们先定义数据实体 Person 和枚举类型 PersonStatus

```java
@Getter
@Setter
@JsdTable(nameStyle = NameStyle.LOWER)
public static class Person {
    @Id
    @GeneratedValue
    private int id;
    private String name;
    private int age;
    private PersonStatus status;
    private LocalDateTime time;
}

public enum PersonStatus implements EnumValueSupport {
    VALID(1), INVALID(2);

    private int value;

    PersonStatus(int value) {
        this.value = value;
    }

    @Override
    public int value() {
        return value;
    }
}
```

同时为了简化代码书写, 我们使用 static 方式 import 如下类型

```java
import static lark.db.sql.SqlHelper.*;
import static lark.db.sql.FilterType.*;
import static lark.db.sql.SortType.*;
```

## 初始化 SqlQuery 对象

在使用 **jsd** 前，我们首先需要创建一个 **SqlQuery** 对象。

```java
public class TestDao {
    private SqlQuery query;

    @Autowired
    public TestDao(DataSource ds) {
        this.query = new SqlQuery(ds);
    }    
}
```

## 插入

### 插入单条记录

```java
InsertResult result = query.insert("person").columns("id", "name").values(1, "n1").result();
System.out.println(String.format("{affectedRows: %d, keys: %s}", result.getAffectedRows(), result.getKey().intValue()));
```

### 插入多条记录

```java
InsertResult result = query.insert("person").columns("id", "name").values(1, "n1").values(2, "n2").result();
System.out.println(String.format("{affectedRows: %d}", result.getAffectedRows()));
```

### 插入 Java 实体对象

```java
Person p = new Person();
p.setId(22);
p.setName("abc");

InsertResult result = query.insert(p).result();
System.out.println(String.format("{affectedRows: %d, keys: %s}", result.getAffectedRows(), result.getKey().intValue()));
```

> 注意: 属性上的 `@GeneratedValue` 注解表示此属性的值将由数据库自增生成, 在插入对象时此属性会自动跳过

### 插入多条记录并获取自增 ID

```java
InsertResult result = query.insert("person").columns("id", "name").values(1, "n1").values(2, "n2").result(true);
System.out.println(String.format("{keys: %s}", result.getKeys()));
```

> 注意: 受限于数据库驱动实现, 部分数据库在插入多行记录时也只会返回单个自增 ID (比如 SQL Server)

## 查询

### 查询单条记录

直接返回数据

```java
SelectResult result = query.select("Id", "Name", "Age").from("person").where(f("id", 22)).result();
Map<String, Object> row = result.one();
```

返回 Java 对象

```java
SelectResult result = query.select("Id", "Name", "Age").from("person").where(f("id", 22)).result();
Person p = result.one(Person.class);
```

### 查询数量

```java
long count = (long)query.select(count()).from("person").result().value();
```

> 注意: 返回的 **count** 数据类型与数据库驱动有关, MySQL 是 `long`, SQL Server 是 `int`

### 查询多条记录

```java
SelectResult result = query.select("Id", "Name", "Age").from("person").where(f("id", 22)).result();
List<Person> persons = result.all(Person.class);
```

### 分页

```java
List<Person> orders = query.select(cs("Id,Name,Age").where(f).page(pageIndex, pageSize).result().all(Person.class);
```

### 根据对象自动生成查询列

```java
SelectResult result = query.select(Person.class).where(f("id", 22)).result();
Person p = result.one(Person.class);
```

### 流式读取

当查询大批量数据进行处理时, 为防止内存溢出, 可以用流式方式进行读取处理

```java
SelectResult result = query.select("Id", "Name", "Age").from("person").where(f("id", GT, 22)).result();
result.each(reader -> println("each: " + reader.getInt(1)));
```

### IN 查询条件

**jsd** 对 IN 查询条件做了优化处理, 支持直接传入数组, 

```java
SelectResult result = query.select("Id", "Name", "Age").from("person").where(f("id", IN, new int[]{19, 20})).result();
List<Person> persons = result.all(Person.class);
```

### 复杂查询

示例 SQL

```sql
SELECT ID, NAME_CN, COUNT(*) AS Count 
FROM Activity AS A 
JOIN ActivityCategory AS AC ON A.ActivityCategory=AC.ID 
WHERE A.ENTER_TIME < ? 
GROUP BY AC.ID, AC.NAME_CN 
HAVING Count > ? 
ORDER BY A.ActivityCategory ASC 
LIMIT 0, 10
```

**jsd** 实现

```java
Table t1 = t("Activity", "A");
Table t2 = t("ActivityCategory", "AC");
Date date = new SimpleDateFormat("yyyy-MM-dd").parse("2014-11-20");
SelectResult result = query.select(c(t2, "ID", "NAME_CN").add("COUNT(*)", "Count"))
        .from(t1).join(t2, f(t1, "ActivityCategory", t2, "ID"))
        .where(f(t1, "ENTER_TIME", LT, date))
        .groupBy(g(t2, "ID", "NAME_CN"))
        .having(f("Count", GT, 0))
        .orderBy(s(ASC, t1, "ActivityCategory"))
        .limit(0, 10)
        .result();
```

## 更新

### 直接更新

```java
// 更新 id = 1 或 id = 2 的记录
UpdateValues values = uv("name", "new name");
Filter f = f("id", 1).or(f("id", 2));
SimpleResult result = query.update("person").set(values).where(f).result();
System.out.println(String.format("{affectedRows: %d}", result.getAffectedRows()));
```

### Java 对象方式

```java
Person p = new Person();
p.setId(22);
p.setName("abc");

// 只更新 name 列
SimpleResult result = query.update(p, "name").result();
System.out.println(String.format("{affectedRows: %d}", result.getAffectedRows()));
```

> 注意: 用 Java 对象方式更新列时, 条件为对象带有 `@Id` 注解的属性

## 删除

### 直接删除

```java
// 删除 id = 1 的记录
SimpleResult result = query.delete("person").where(f("id", 1));
System.out.println(String.format("{affectedRows: %d}", result.getAffectedRows()));
```

### Java 对象方式

```java
// 删除 id = 1 的记录
Person p = new Person()
p.setId(1);

SimpleResult result = query.delete(p).result();
System.out.println(String.format("{affectedRows: %d}", result.getAffectedRows()));
```

## 直接执行 SQL 语句

```java
try (ExecuteResult result = query.execute("select * from person where id<?", 100).result()) {
    Person p = result.one(Person.class);
}
```

> 注意: 使用 `ExecuteResult` 必须显示调用 close 方法关闭连接，其实现了 Java 7 AutoCloseable 接口，因此可以用 try 块包含自动释放连接

## 事务

**jsd** 完全兼容 Spring 的事务注解，如果你想，你甚至可以跟 MyBatis 或 JPA 混用。