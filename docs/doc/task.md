# 计划任务

定时执行某个任务是实际开发中最常用的功能之一，**lark-task** 中封装了计划任务相关组件，基于 **xxl-job** 进行调度。

## 开发 执行器

使用 **lark-cli** 工具来创建 **计划任务** 项目模块, 它会帮你生成一个完整而规范的项目骨架.

第一步: 创建项目(如果是已经存在的项目, 可以跳过这一步)

```bash
lark new project -group lark lark-example
```
> Usage: lark new project -group ${groupName} -artifact ${artifactName} ${projectDirname}

第二步: 创建 **计划任务** 模块

```bash
cd lark-example
lark new task -group lark -artifact lark-example-task task
```
> Usage: lark new task -group ${groupName} -artifact ${artifactName} ${moduleDirName}

通过这两个步骤就创建了一个基于 **lark-task** 框架实现的任务执行器模块, 打开模块的 `pom.xml` 文件, 可以看到此模块是从 **lark-starter-task** 派生的

```xml
<parent>
    <relativePath></relativePath>
    <groupId>lark</groupId>
    <artifactId>lark-starter-task</artifactId>
    <version>1.5.0-SNAPSHOT</version>
</parent>
```

同时应用程序入口类改成了 **TaskApplication**.

```java
@SpringBootApplication
public class Bootstrap {
    public static void main(String[] args) {
        TaskApplication app = new TaskApplication(Bootstrap.class);
        app.run();
    }
}
```

在 application.yml 中存在如下配置

```yaml
lark:
  task:
    address: http://127.0.0.1:8080/xxl-job-admin
    token: 123456
    executorUrl: http://techwis-task-qa.sanqlt.com/lark-example-task/
```

> 备注：executorUrl不配置则默认为当前的IP+端口

## 开发 执行任务

**lark-cli** 会自动生成一个示例用的任务 TestTask，你可以修改它，完成你的真正业务处理逻辑。

> **TaskApplication** 会自动扫描包内所有实现了 **Task** 注解的类
> 
```java
@Component
@Task
public class TestTask implements Executor {
    
    @Override
    public void execute(TaskContext ctx) {
        int[] userIds = new int[]{123,1,0,124};
        for ( int userId : userIds ) {
            ctx.info( "===> UserId:{}", userId );
        }
    }
}
```

> 备注：需要在管理后台查看的执行日志需要通过ctx.info记录，每个任务只保留最新一次 执行日志。

## 后台配置（XXL-JOB）

### 配置执行器

每一个Task对应Xxl-job的一个执行器，执行器需要在 **Xxl-Job** 管理后台（`执行器管理`）中进行配置。

* 必要属性说明

| 属性   | 说明                                       | 示例值                        |
| ---- | ---------------------------------------- | -------------------------- |
| AppName   | 执行器（Task项目）名称，必须唯一    | 如: lark-example-task        |
| 名称   | 执行器（Task项目）标题 | 如: 测试执行器                 |
| 注册方式   | 执行器（Task项目）启动后，会定时上报地址进行自动注册 | 默认值：自动注册                 |

### 配置任务

每一个Task中可以包含多个任务, 每一个任务可以配置为不同的定时任务, 定时任务需要在 **Xxl-Job** 管理后台（`任务管理`）进行配置。

* 必要属性说明

| 属性   | 说明                                       | 示例值                        |
| ---- | ---------------------------------------- | -------------------------- |
| 执行器   | 执行器的标题    | 如: 测试执行器        |
| Cron   | 任务执行时间的 cron 表达式 | 如：0/30 * * * * ?                  |
| JobHandler  | 任务（Task）名称                                      | 如: TestTask |
| 任务参数  | 执行任务时需要的额外参数，参数形式：name1:value1&#124;name2:value2| 如: id:1&#124;name:2|

## 常见问题