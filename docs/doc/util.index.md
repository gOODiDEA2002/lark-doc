# 索引服务

lark.util.Index.IndexService 封装了对ElasticSearch的常用方法

## 场景

### 后台
提供列表操作查询

### 前台
提供搜索支持

## 快速上手

### POM

在 pom.xml 中添加如下依赖

```xml
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-util-index</artifactId>
</dependency>
```

### 配置

在 application.yml 中添加如下配置

```yaml
lark:
  util:
    index:
      address: 192.168.99.92:9200
```

### 定义索引中存放的对象文档（Document）

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserDocument {

    private int id;

    private String name;

    private String mobile;
}
```

### 定义索引服务（Index）

```java
@Service
public class UserIndexService {
    /**
     * 索引名称，一般为：项目名.模块名.索引名
     */
    private static final String INDEX_NAME = "lark.example.index.user";
    private static final int EXCEPTION_CODE = 10009;
    /**
     * 注入索引服务
     */
    @Autowired
    IndexService indexService;

    /**
     * 添加用户进索引
     *
     * @param user 用户信息
     */
    public void add(UserDocument user) {
        IndexDocument<UserDocument> document = new IndexDocument<>();
        document.setId(String.valueOf(user.getId()));
        document.setData(user);
        try {
            indexService.save(INDEX_NAME, document);
        } catch (IOException e) {
            throw new BusinessException(EXCEPTION_CODE, "Failed to add user index", e);
        }
    }

    /**
     * 根据ID获取用户文档
     *
     * @param userId 用户 ID
     * @return 用户对象
     */
    public UserDocument getUser(int userId) {
        try {
            UserDocument userEntity = indexService.get(INDEX_NAME, String.valueOf(userId), UserDocument.class);
            return userEntity;
        } catch (IOException e) {
            throw new BusinessException(EXCEPTION_CODE, "Failed to get user index", e);
        }
    }

    /**
     * 根据姓名获取用户文档列表
     *
     * @param name 用户姓名
     * @return 用户集合
     */
    public List<UserDocument> getUsersByName(String name) {
        // 构建查询条件
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery()
                .must(QueryBuilders.matchPhraseQuery("name", name));
        // 构建查询生成器
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder().query(boolQueryBuilder);
        try {
            List<UserDocument> userEntityList = indexService.searchByQuery(INDEX_NAME, sourceBuilder, UserDocument.class);
            return userEntityList;
        } catch (IOException e) {
            throw new BusinessException(EXCEPTION_CODE, "Failed to get user index", e);
        }
    }

    /**
     * 根据用户I删除用户文档
     *
     * @param userId 用户 ID
     */
    public void deleteUser(int userId) {
        try {
            indexService.delete(INDEX_NAME, String.valueOf(userId));
        } catch (IOException e) {
            throw new BusinessException(EXCEPTION_CODE, "Failed to delete user index", e);
        }
    }
}
 ```

### 应用

 ```
    @Autowired
    UserIndexService userIndexService;
    ……
    UserDocument user = new UserDocument( userId, userName, mobile );
    userIndexService.save( user );
    ……
    user = userIndexService.getUser( userId );
    ……
 ```

## 常见问题


