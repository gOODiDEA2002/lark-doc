# API

API是面向客户端（如：iOS，Android，H5等）提供http接口的模块

Api分为 **契约** 和 **服务** 两种模块

## API契约

使用 **lark-cli** 工具来创建 **API契约** 模块, 它会帮你生成一个完整而规范的项目骨架.

第一步: 创建项目(如果是已经存在的项目, 可以跳过这一步)

```bash
lark new project -group lark lark-example
```
> Usage: lark new project -group ${groupName} -artifact ${artifactName} ${projectDirname}

第二步: 创建 **API契约** 模块

```bash
cd lark-example
lark new api-contract -group lark -artifact lark-example-api-contract api-contract
```
> Usage: lark new api-contract -group ${groupName} -artifact ${artifactName} ${moduleDirName}

通过这两个步骤就创建了一个 **API契约** 模块, 打开模块的 `pom.xml` 文件, 可以看到此模块是从 **lark-starter-api-contract** 派生的

```xml
<parent>
    <relativePath></relativePath>
    <groupId>lark</groupId>
    <artifactId>lark-starter-api-contract</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```

我们需要在此模块中定义接口及接口的请求、响应相关对象

> 备注：后面我们会通过此模块提供Mock服务

* 定义接口数据对象

```java
public class TestVo {
    /**
     * 请求参数
     */
    @Data
    @NoArgsConstructor
    @EqualsAndHashCode
    public static class HelloRequest {
        /**
         * ID
         */
        private int id;
        /**
         * 姓名
         */
        private String name;
    }

    /**
     * 响应结果
     */
    @Data
    @NoArgsConstructor
    @EqualsAndHashCode
    public static class HelloResponse {
        /**
         * 结果
         */
        private String result;
        /**
         * 时间
         */
        private long time;
    }
}

```

* 定义接口

```java
@RestController
@RequestMapping("/test")
@Api(tags="测试接口")
public interface TestApi {

    @ApiOperation("Hello")
    @ApiResponse(responseCode="200", description="Hello")
    @PostMapping("/hello.api")
    public HelloResponse hello(@ApiParam("HelloRequest") HelloRequest hello );
}
```

## API服务
同样使用 **lark-cli** 工具来创建 **API** 模块

```bash
cd lark-example
lark new api -group lark -artifact lark-example-api api
```
> Usage: lark new api -group ${groupName} -artifact ${artifactName} ${moduleDirName}

打开模块的 `pom.xml` 文件, 可以看到此模块是从 **lark-starter-api** 派生的 且 依赖 **api-contract**

```xml
<parent>
    <relativePath></relativePath>
    <groupId>lark</groupId>
    <artifactId>lark-starter-api</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
...
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-example-api-contract</artifactId>
    <version>&#36;{project.version}</version>
</dependency>
```

同时应用程序入口类改成了 **ApiApplication**.

```java
@SpringBootApplication
public class Bootstrap {
    public static void main(String[] args) {
        ApiApplication app = new ApiApplication(Bootstrap.class);
        app.run();
    }
}
```

* 实现API契约中定义的接口

```java
@RestController
public class TestController implements TestApi {
    @Autowired
    TestBiz testBiz;
    
    @Override
    public TestVo.HelloResponse hello(TestVo.HelloRequest hello) {
        if ( hello.getId() == 0 ) {
            throw new ApiFaultException( 1000, "ID must >= 0" );
        }
        if (Strings.isEmpty( hello.getName() ) ) {
            throw new ApiFaultException( 1001, "Name is empty" );
        }
        return testBiz.hello( hello );
    }
}
```

* 响应内容

**lark-api** 会将所有API响应进行一层封装，以便于客户端更好处理

    - code: 成功：0，错误可以通过ApiFaultException进行自定义
    - msg: 成功：""
    - data: 返回的具体数据

```json
{
  "code":0,
  "msg":"",
  "data": 
    { "result":"456","time":1611744388992}
}
```

## 常见问题