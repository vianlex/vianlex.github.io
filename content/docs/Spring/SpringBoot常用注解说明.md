---
title: "SpringBoot 常用注解说明"
linkTitle: "SpringBoot 常用注解说明"
weight: 20
---
## @Controller 和 @RestController

在 `Spring Boot` 中，`@Controller` 和 `@RestController` 都是用于处理 `HTTP` 请求的注解，它们的主要区别是默认返回响应类型不同。

### @Controller

主要用于返回视图（`HTML`页面），如果想要能返回 `JSON/XML`数据需要配合 `@ResponseBody` 注解。

注意：SpringBoot 返回 JSP、Freemarker 等视图时，需要先配置视图解析类 `ViewResolver`。

```java

@Controller
@RequestMapping("/web")
public class WebController {

    // 返回视图（HTML页面）
    @GetMapping("/home")
    public String home() {
        // 返回视图名，对应 home
        return "home";
    }

    // 返回 HTML 页面
    @GetMapping("/index")
    public String index(Model model) {
        model.addAttribute("message", "Hello World");
        // 返回 index 视图
        return "index";
    }

    @RequestMapping("/about")
    public ModelAndView about() {
        ModelAndView modelAndView = new ModelAndView();
        // 指定返回的视图名称，返回 about.html
        modelAndView.setViewName("about");
        // 设置数据
        modelAndView.addObject("message", "Hello, About!");
        // 返回 ModelAndView 对象
        return modelAndView;
    }


    @GetMapping("/data")
    // 需要@ResponseBody才能返回JSON
    @ResponseBody
    public User getData() {
        return new User("John", 25);
    }
}

```

### RestController

`@RestController` 是 `@Controller` 和 `@ResponseBody` 的组合，默认所有方法都返回 JSON/XML 数据，而不是视图。

```java

@RestController
@RequestMapping("/api")
public class ApiController {

    // 自动返回JSON数据
    @GetMapping("/user")
    public User getUser() {
        // 自动转换为JSON
        return new User("John", 25);
    }

    @PostMapping("/create")
    public ResponseEntity<?> createUser(@RequestBody User user) {
        return ResponseEntity.ok(user);
    }
}

```

## @Qualifier

当容器中存在多个同类型的 Bean 时，`@Autowired` 默认按类型（byType）注入会因找不到唯一候选而报错。此时配合 `@Qualifier` 按名称（byName）精确指定要注入的 Bean。

```java
@Service
public class OrderService {

    // 存在多个 Payment 实现时，指定注入名称为 alipayPayment 的 Bean
    @Autowired
    @Qualifier("alipayPayment")
    private Payment payment;
}
```

多个实现 Bean 的定义示例：

```java
@Configuration
public class PaymentConfig {

    @Bean("alipayPayment")
    public Payment alipayPayment() {
        return new AlipayPayment();
    }

    @Bean("wechatPayment")
    public Payment wechatPayment() {
        return new WechatPayment();
    }
}
```

## @Autowired

按类型自动装配依赖。可标注在字段、构造方法、Setter 方法上。Spring 4.3+ 起，当类只有一个构造方法时，可以省略 `@Autowired`。

```java
@Service
public class UserService {

    // 字段注入
    @Autowired
    private UserMapper userMapper;

    private final OrderMapper orderMapper;

    // 构造器注入（推荐：依赖不可变，便于测试）
    public UserService(OrderMapper orderMapper) {
        this.orderMapper = orderMapper;
    }

    private CacheService cacheService;

    // Setter 注入
    @Autowired
    public void setCacheService(CacheService cacheService) {
        this.cacheService = cacheService;
    }
}
```

`required = false` 表示依赖可选，容器中不存在对应 Bean 时也不报错：

```java
@Autowired(required = false)
private OptionalService optionalService;
```

## @Value

从配置文件（application.yml / properties）或环境变量中注入值，支持默认值与 SpEL 表达式。

```java
@Component
public class AppConfig {

    // 注入字符串，冒号后为默认值
    @Value("${app.name:default-app}")
    private String appName;

    // 注入数字
    @Value("${app.max-size:100}")
    private int maxSize;

    // SpEL 表达式
    @Value("#{systemProperties['user.home']}")
    private String userHome;

    // 注入数组（逗号分隔）
    @Value("${app.servers:}")
    private List<String> servers;
}
```

## @ConfigurationProperties

批量把配置文件中同前缀的一组属性绑定到一个 POJO 上，比逐字段 `@Value` 更整洁。

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private int maxSize;
    private List<String> servers;

    // 省略 getter / setter
}
```

对应配置：

```yaml
app:
  name: my-app
  max-size: 100
  servers:
    - server1
    - server2
```

也可以不在类上加 `@Component`，改用 `@EnableConfigurationProperties` 注册：

```java
@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)
public class Application {
}
```

## @SpringBootApplication

Spring Boot 启动类核心注解，是三个注解的合成：

- `@SpringBootConfiguration`：标记当前类为配置类；
- `@EnableAutoConfiguration`：开启自动装配；
- `@ComponentScan`：扫描当前包及其子包下的组件。

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

如需排除某些自动装配，可用 `exclude` 属性：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class Application {
}
```

## @Component 家族

都是把类交给 Spring 容器管理，语义上的区别便于分层阅读：

- `@Component`：通用组件；
- `@Service`：业务逻辑层；
- `@Repository`：数据访问层，还会把持久层异常转换为 Spring 的 `DataAccessException`；
- `@Controller`：控制层（见前文）。

```java
@Component
public class CommonUtil { }

@Service
public class UserService { }

@Repository
public class UserDao { }
```

## @RequestMapping 家族

用于映射 HTTP 请求，可标注在类或方法上。`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`、`@PatchMapping` 是其按方法简化的组合注解。

```java
@RestController
@RequestMapping("/users")
public class UserController {

    // GET /users/123
    @GetMapping("/{id}")
    public User getUser(@PathVariable("id") Long id) {
        return userService.getById(id);
    }

    // GET /users?name=Tom&page=1
    @GetMapping
    public List<User> list(@RequestParam String name,
                           @RequestParam(defaultValue = "1") int page) {
        return userService.list(name, page);
    }

    // POST /users，请求体为 JSON
    @PostMapping
    public User create(@RequestBody User user) {
        return userService.save(user);
    }

    // PUT /users/123
    @PutMapping("/{id}")
    public User update(@PathVariable Long id, @RequestBody User user) {
        return userService.update(id, user);
    }

    // DELETE /users/123
    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

其他常用参数注解：

- `@RequestHeader`：读取请求头；
- `@CookieValue`：读取 Cookie；
- `@ModelAttribute`：绑定表单数据到模型对象。

## @Transactional

声明式事务，标注在类或方法上。默认只对 `RuntimeException` 及其子类（未受检异常）回滚，受检异常需用 `rollbackFor` 指定。

```java
@Service
public class OrderService {

    @Transactional
    public void createOrder(Order order) {
        orderMapper.insert(order);
        // 抛出 RuntimeException 时自动回滚
        stockService.deduct(order.getSkuId(), order.getCount());
    }

    // 指定受检异常也回滚；readOnly 表示只读事务
    @Transactional(rollbackFor = Exception.class, readOnly = true)
    public Order getById(Long id) {
        return orderMapper.selectById(id);
    }
}
```

注意：`@Transactional` 依赖 Spring 的代理机制，同类内部方法直接调用不会触发事务，需通过注入自身或拆分类来解决。

## @Scheduled 与 @Async

`@Scheduled` 用于定时任务，需在启动类加 `@EnableScheduling`；`@Async` 用于异步方法，需加 `@EnableAsync`。

```java
@SpringBootApplication
@EnableScheduling
@EnableAsync
public class Application {
}

@Component
public class TaskService {

    // 固定速率：上次开始执行后 5 秒再执行
    @Scheduled(fixedRate = 5000)
    public void report() {
        System.out.println("每 5 秒执行一次");
    }

    // 固定延迟：上次执行结束后 5 秒再执行
    @Scheduled(fixedDelay = 5000)
    public void sync() { }

    // Cron 表达式：每天凌晨 2 点执行
    @Scheduled(cron = "0 0 2 * * ?")
    public void dailyJob() { }

    // 异步执行，不阻塞主线程
    @Async
    public void sendMessage(String content) { }
}
```

## @ControllerAdvice 与 @ExceptionHandler

全局异常处理：`@ExceptionHandler` 捕获指定异常，`@ControllerAdvice` 使其作用于所有控制器（`@RestControllerAdvice` 是其返回 JSON 的版本）。

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<?> handleBusiness(BusinessException e) {
        return ResponseEntity.badRequest().body(e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handleOther(Exception e) {
        return ResponseEntity.internalServerError().body("系统异常");
    }
}
```

## 参数校验注解

配合 `@Valid`（或 `@Validated`）触发校验，常用注解均来自 Jakarta Validation（`jakarta.validation.constraints`）：

- `@NotNull`：不能为 null；
- `@NotBlank`：字符串不能为 null 且去除首尾空格后长度大于 0；
- `@NotEmpty`：集合、数组、字符串不能为空；
- `@Size(min, max)`：限定长度或元素个数；
- `@Min` / `@Max`：限定数值范围；
- `@Pattern`：正则匹配。

```java
@RestController
public class UserController {

    @PostMapping("/users")
    public User create(@Valid @RequestBody UserDTO dto) {
        return userService.save(dto);
    }
}

public class UserDTO {

    @NotBlank(message = "用户名不能为空")
    @Size(min = 2, max = 20)
    private String name;

    @NotNull
    @Min(value = 1, message = "年龄必须大于 0")
    private Integer age;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;

    // 省略 getter / setter
}
```