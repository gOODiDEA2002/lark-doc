# Service

Service是最常用的模块类型，对其它模块提供服务（Http协议）

Service分为 **契约** 和 **服务** 两种模块

## Service契约

使用 **lark-cli** 工具来创建 **Service契约** 模块, 它会帮你生成一个完整而规范的项目骨架.

第一步: 创建项目(如果是已经存在的项目, 可以跳过这一步)

```bash
lark new project -group lark lark-example
```
> Usage: lark new project -group ${groupName} -artifact ${artifactName} ${projectDirname}

第二步: 创建 **Service契约** 模块

```bash
cd lark-example
lark new service-contract -group lark -artifact lark-example-service-contract service-contract
```
> Usage: lark new service-contract -group ${groupName} -artifact ${artifactName} ${moduleDirName}

通过这两个步骤就创建了一个 **Service契约** 模块, 打开模块的 `pom.xml` 文件, 可以看到此模块是从 **lark-starter-service-contract** 派生的

```xml
<parent>
    <relativePath></relativePath>
    <groupId>lark</groupId>
    <artifactId>lark-starter-service-contract</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```

我们需要在此模块中定义接口及接口的请求、响应相关对象

> 备注：其它模块需引用此模块来调用具体的Service服务

* 定义接口数据对象

```java
public class GetOrderDto {
    /**
     * 请求参数
     */
    @Data
    @NoArgsConstructor
    @EqualsAndHashCode
    public static class GetOrderRequest {
        /**
         * ID
         */
        private long orderId;
    }

    /**
     * 响应结果
     */
    @Data
    @NoArgsConstructor
    @EqualsAndHashCode
    public static class GetOrderResponse {
        /**
         * 订单ID
         */
        private long orderId;
        /**
         * 用户ID
         */
        private long userId;

        /**
         * SKUID
         */
        private int skuId;
    }
}
```

* 定义接口

```java
@RestController
@RequestMapping("/test")
public interface TestService {
    
    @PostMapping( "/order/get.srv")
    GetOrderDto.GetOrderResponse getOrder(@RequestBody GetOrderDto.GetOrderRequest request);
}
```

## Service服务
同样使用 **lark-cli** 工具来创建 **Service** 模块

```bash
cd lark-example
lark new service -group lark -artifact lark-example-service service
```
> Usage: lark new service -group ${groupName} -artifact ${artifactName} ${moduleDirName}

打开模块的 `pom.xml` 文件, 可以看到此模块是从 **lark-starter-service** 派生的 且 依赖 **service-contract**

```xml
<parent>
    <relativePath></relativePath>
    <groupId>lark</groupId>
    <artifactId>lark-starter-service</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
...
<dependency>
    <groupId>lark</groupId>
    <artifactId>lark-example-service-contract</artifactId>
    <version>&#36;{project.version}</version>
</dependency>
```

同时应用程序入口类改成了 **ServiceApplication**.

```java
@SpringBootApplication
public class Bootstrap {
    public static void main(String[] args) {
        ServiceApplication app = new ServiceApplication(Bootstrap.class);
        app.run();
    }
}
```

* 实现Service契约中定义的接口

```java
@Service("测试服务")
public class TestServiceImpl implements TestService {
    private static final Logger LOGGER = LoggerFactory.getLogger(TestServiceImpl.class);

    @Autowired
    private OrderBiz orderBiz;

    @Override
    public GetOrderDto.GetOrderResponse getOrder(GetOrderDto.GetOrderRequest request) {
        GetOrderDto.GetOrderResponse response = new GetOrderDto.GetOrderResponse();
        //
        long orderId = request.getOrderId();
        LOGGER.info("request-> orderId:{}", orderId);
        OrderDO order = orderBiz.getOrder(orderId);
        if (order != null) {
            LOGGER.info("response-> orderId:{}, userId:{}, skuId:{}", order.getOrderId(), order.getUserId(), order.getSkuId());
            response.setOrderId(order.getOrderId());
            response.setUserId(order.getUserId());
            response.setSkuId(order.getSkuId());
        } else {
            LOGGER.info("response-> order not found");
        }
        return response;
    }
}
```

## 常见问题