# Sentinel学习笔记.md


## Sentinel 核心概念

- 资源：资源可以是任何东西，服务，服务里的方法，甚至是一段代码。
- 规则：设定保证资源的实时规则，可以包括流量控制规则、熔断降级规则以及系统保护规则。所有规则可以动态实时调整，

## Sentinel 的作用

使用 Sentinel 主要为了提高系统运行期间的稳定性和可用性。

在微服务环境下，服务之间存在复杂的调用关系，单个服务的故障或过载可能会迅速影响到整个系统，导致服务雪前效应，Sentinel 的流控组件可以限制进入系统的流量，防止系统因超出处理能力而前溃，Sentinel 的降级组件则在服务不可用或响应过慢时，提供降级逻辑，如返回备用数据或执行降级操作，以保证核心业务的正常运行。


## @SentinelResource 注解使用

第一步引入依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
    <version>1.8.6</version>
</dependency>

<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
    <version>1.8.6</version>
</dependency>
<!-- SentinelResourceAspect 实现在该依赖中-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>1.8.6</version>
</dependency>
```


第二步配置和编写测试

```java
// 配置类
@Configuration
public class SentinelAspectConfiguration {
    @Bean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }
}


// 测试类
@RestController
public class HelloController { 

/**
     * 注解方式，注意要将 SentinelResourceAspect 注入容器中
     * 注意通过方法指定 handler 和 fallback 时，方法要写在同一个类中
     * blockHandler 表示处理限流后抛出 BlockException 后的处理方法
     * fallback 表示处理所有异常的，回调方法
     * 具体可以查看 SentinelResourceAspect 源码
     *
     * @return
     */
    @SentinelResource(value = "helloWorld", blockHandler = "handleException", fallback = "fallbackException")
    @GetMapping(value = "/helloWorld")
    public String helloWorld(@RequestParam(required = false) String name) {
        // 抛出非 BlockException 会执行 fallback 回调
        int i = 1 / 0;
        return "Hello world " + name;
    }


    /**
     * Sentinel 限流后，会抛出 BlockException， 具体可以查看 SentinelResourceAspect 源码
     * Block 异常处理函数，注意：参数必须与资源一致，并且需要添加一个的参数 BlockException
     */
    public String handleException(String name, BlockException ex) {
        return "HelloWorld " + name + " 接口被流控了";
    }

    /**
     * Fallback 异常处理函数, 注意：参数必须与资源一致，并且需要添加一个而外的固定参数 Throwable
     * 表示降级处理资源方法的所有异常，具体可以查看 SentinelResourceAspect 源码
     *
     * @param name
     * @param t
     * @return
     */
    public String fallbackException(String name, Throwable t) {
        return "HelloWorld " + name + " 接口被异常降级了";
    }


    @PostConstruct
    public void initFlowRules() {
        List<FlowRule> rules = new ArrayList<>();
        // helloWorld 资源规则
        FlowRule rule = new FlowRule();
        rule.setResource("helloWorld");
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        // Set limit QPS to 1.
        rule.setCount(1);
        rules.add(rule);
        FlowRuleManager.loadRules(rules);
    }
}

```


## Sentinel 规则动态更新原理

Sentinel 动态更新规则是通过推送的方式实现，即在控制台修改规则后 Sentinel 会将规则推送到服务应用中，如下图。

![Sentinel 规则动态更新原理图](../images/Sentinel规则动态更新原理图.png)


## 微服务整合 Sentinel

第一步引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

第二步添加配置

```yml
spring:
  cloud:
    sentinel:
      # 启用限流，默认通过 AbstractSentinelInterceptor 拦截器，去拦截 SpringMVC 请求进行限流
      # Restemplate 和 OpenFeign 使用 Sentinel 需要添加额外的对应配置 
      enabled: true
      transport: 
        # 添加sentineI的控制台地址
        dashboard: 127.0.0.1:8888
        # 指定微服务应用与 Sentinel 控制台交互的端口，微服务应用会起一个 HttpServer 占用该端口，等待 Sentinel 推送更新的规则
        # 通过 127.0.0.1:8719/api 可以看到本地 Sentinel 客户端有哪些 API 
        port: 8719
```

### 限流统一异常管理

Sentinel 通过  AbstractSentinelInterceptor 拦截器的 preHandle 方法拦截请求，执行 `SphU.entry` 资源保护方法来实现限流的，如果 `SphU.entry` 方法抛出限流异常 `BlockException`，拦截器会捕获该限流异常，并调用 `handleBlockException` 方法获取 `BlockExceptionHandler` 实例对异常进行处理，故我们可以通过自定义实现 BlockExceptionHandler 接口统一处理 `BlockException` 来达到限流统一异常管理。


```java
@Component
public class CustomBlockExceptionHandler implements BlockExceptionHandler {

     @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        // 1. 记录日志
        logBlockException(request, e);
        // 2. 构建响应
        Map<Object, Object> result = buildErrorResult(e, request);
        // 3. 设置响应头
        response.setStatus(Integer.parseInt(result.get("code").toString()));
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=utf-8");
        // 4. 写入响应
        try (PrintWriter writer = response.getWriter()) {
            writer.write(objectMapper.writeValueAsString(result));
            writer.flush();
        }

    }

    private void logBlockException(HttpServletRequest request, BlockException e) {
        String resource = e.getRule() != null ? e.getRule().getResource() : "unknown";
        logger.warn("【Sentinel Block】应用: {}, 资源: {}, URI: {} {}, 异常: {}",
                applicationName, resource, request.getMethod(), request.getRequestURI(), e.getClass().getSimpleName());
    }

   private Map<Object, Object> buildErrorResult(BlockException e, HttpServletRequest request) { 
Map<Object, Object> result = new HashMap<>();
        if (e instanceof FlowException) {
            result.put("code", 429);
            result.put("message", "请求过于频繁，请稍后重试");
        } else if (e instanceof DegradeException) {
            DegradeRule rule = (DegradeRule) e.getRule();
            result.put("code", 429);
            result.put("message", "服务暂时不可用");
        } else if (e instanceof ParamFlowException) {
            ParamFlowException pfe = (ParamFlowException) e;
            result.put("code", 429);
            result.put("message", "热点参数限流");
        } else if (e instanceof SystemBlockException) {
            result.put("code", 503);
            result.put("message", "系统保护模式");
        } else if (e instanceof AuthorityException) {
            result.put("code", 403);
            result.put("message", "访问被拒绝");
        } else {
            result.put("code", 429);
            result.put("message", "系统限流");
        }                              
   }
}

```



### 通过 Sentinel 对外暴露的 Endpoint 查看流控规则

第一步引入依赖

```xml
<dependency>
    <groupId>org-springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

第二步在配置文件中增加以为配置

```yml
# 暴露 actuator 端点
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

第三步访问地址 `htpp://微服务地址端口/actuator/sentinel` 即可查看 Sentinel 配置的流控规则信息



## 流控效果

1、快速失败（直接拒绝）

- 达到调值后，新的请求会被立即拒绝并抛出 FlowException 异常，是默认的处理方式。
- 适用场景：短时突发流量瞬时高峰需要立即拒绝、非核心业务可以承受一定比例的请求失败、压力测试过明确知道系统的承载上限

2、Warm Up(预热)

- 预热模式，对超出间值的请求同样是拒绝并抛出异常。但是根据阀值会动态变化，从一个较小值逐渐增加到最大调值。
- 预热过程，计算过程
    ```java
      int finalQps = 100;           // 最终阈值(Sentinel 设置的 QPS 阈值)
      int warmUpSeconds = 600;      // 预热时间(Sentinel 设置的预热时间)
      int coldFactor = 3;           // 冷启动因子(coldFactor 默认为3，可以通过系统参数调整)
      // 初始阈值 = 最终阈值 / coldFactor
      int initialQps = finalQps / coldFactor;  // 33 QPS
      // 每秒增长的QPS
      double qpsPerSecond = (finalQps - initialQps) / (double) warmUpSeconds;
      // === Warm Up 过程模拟 ===
      // 初始阈值: 33 QPS
      // 预热时间: 600 秒
      // 每秒增长: 0.112 QPS 
      // 第  0秒: 33.0 QPS （开始访问 100 个请求 67 个拒绝，33 个通过）
      // 第 60秒: 39.7 QPS （60s 后 100 个请求 60.3 个拒绝，39.7 个通过）
      // 第300秒: 66.7 QPS
      // 第600秒: 100.0 QPS
      // 最终阈值: 100 QPS
    ```
- 适用以下场景：长期闲置的服务重启、突发流量保护、定时任务触发大量请求。

3、排队等待（匀速排队）

-  让请求按先后次序排队执行，QPS 阀值设置 10，则排队间隔时间为 1000ms / 10  = 100ms，如果`上一个请求的通过时间 + 请求间隔 - 当前请求到达时间 > 设置的超时时间` 则拒绝请求。 注意匀速排队设置的 QPS 不能大于 1000。
-  适用场景：突发流量削峰填谷，保证处理速率稳定（如第三方 API 调用有频率限制）


## Sentinel 的规则持久化问题

Sentinel 控制台，默

### 默认模式

1. 当微服务启动时，微服务本地的 Sentinel 客户端，会向 Sentinel 控制台注册客户端信息，并推送本地流控规则到 Sentinel 控制台内存中，默认情况下，微服务本地内存中是没有流控规则，故微服务重新启动会导致 Sentinel 中的流控规则丢失。
2. 当 Sentinel 控制台中，修改流控规则时，Sentinel 控制台会通过微服务中的 Sentinel 客户端更新规则接口，将更改后的规则信息推送微服务本地内存中。
3. 微服务正常运行，Sentinel 控制台重启后规则也会丢失（也是缓存在内存中），当微服务访问接口时，才会将微服务本地内存中的规则，推送到 Sentinel 控制台中。
4. Sentinel 控制台更新规则后，最终是通过微服务中的 ModifyRulesCommandHandler 实现类去更新微服务本地内存规则的，期间会判断是否有实现有写数据源 WritableDataSource 实现类，官方默认是没有实现的，想要持久化，我们通过实现改接口来达到目的。  


### 自定义 pull 模式持久化规则

- 官方实现的方案：基于文件实现拉模式，官方 Demo 实现: sentinel-demo/sentinel-demo-dynamic-file-rule
- 基于文件方案：当 Sentinel 控制台推送规则到微服务本地内存时，我们实现 WritableDataSource 将规则写到本地文件中，如果我们想在本地文件中修改规则，可以实现一个定时任务监控本地文件是否发生了变化（比较文件 MD5 判断文件是否改变），如果变化了就将文件中的规则，更新到本地内存中。
- 基于 Nacos 方案：当 Sentinel 控制台推送规则到微服务本地内存时，我们实现 WritableDataSource 将规则发布到 Nacos 配置中心，同时监听 Nacos 的修改更新，将 Naocs 修改改的规则同步到微服务本地内存中。

拉模式的优点：实现简单，无任何依赖，缺点：不能保证实时一致性问题。


## 自定义 push 模式持久化规则

1. Sentinel 控制台将规则送到 Nacos 等配置中心，微服务监听配置中心的规则，当规则发生改变后，将规则更新到微服务本地内存中。




