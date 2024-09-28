# ANT 和 MVC 路径模式笔记

ANT 路径模式和 MVC 路径模式是 Spring 框架中用于匹配 URL 路径的两种模式，它们有不同的语法和用途，主要用于配置 Spring Security 和 Spring MVC 中的请求映射。

## ANT 路径模式

ANT 路径模式，支持的匹配符如下：

- `*` 表示匹配除 `/` 符以外的，任意 0 个或者多个字符
- `**` 表示匹配任意 0 个或者多个字符（包含路径分割 `/` 符）
- `?` 表示匹配除 `/` 符以外的任意一个字符

匹配例子如下

```java

import org.springframework.util.AntPathMatcher;

public class UrlPatternTest {

    public static void main(String[] args) {
        AntPathMatcher antPathMatcher = new AntPathMatcher();
        System.out.println(antPathMatcher.match("/*", "/")); // 输出 true
        System.out.println(antPathMatcher.match("/*", "/hello")); //输出 true

        System.out.println(antPathMatcher.match("/wms/**", "/wms/")); // 输出 true
        System.out.println(antPathMatcher.match("/wms/**", "/wms/goods/index")); // 输出 true

        System.out.println(antPathMatcher.match("/?","/a")); // 输出 true
        System.out.println(antPathMatcher.match("/?", "/aa")); // 输出 false
        System.out.println(antPathMatcher.match("/?", "/")); // 输出 false
    }

}

```


## MVC 路径匹配模式

 MVC 路径模式通常用于 SpringBoot 控制器的请求映射，支持的匹配符号如下：

- `{}` 表示匹配路径变量，如：`/goods/{id}` 可以匹配 `/goods/20240923001`
- `*` 表示匹配除了 `/` 符以外的任意字符（0 或者多个），如：`/goods/*` 可以匹配 `/goods、/goods/list`，注意不匹配 `/goods/`
- `**` 表示匹配任意字符（0 或者多个）包括 `/` 符，如：
