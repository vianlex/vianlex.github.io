# SpringBoot 常用注解说明


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

`@RestController` 是 `@Controller` 和 `@ResponseBody` 的组合，默认所有方法都返回JSON/XML数据，而不是视图。

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

