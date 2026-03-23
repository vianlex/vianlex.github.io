# Spring Security 学习笔记

Spring Security 的核心原理是构建一套安全过滤器链（Filter Chain），对每个进入应用的 HTTP 请求进行安全检查。通过可配置的过滤器（Filter）来实现，将认证（Authentication）、授权（Authorization）、防护（Protection）等功能模块化。




## SpringBoot 整合 Security

### 引入 Spring Security 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>5.7.11</version>
</dependency>
```

### 密码登录模式

引入依赖后，不做任何配置就可以直接使用了，默认使用密码登录的校验模式。当启动项目后，会输出默认的登录密码（默认用户是 user）如下：

```text
Using generated security password: f54d4a30-ebbc-47e2-b0be-6969605cc862
```

通过配置文件修改默认的用户名和用户密码，如下：

```yaml
# 很多开源软件，只有一个用户就是简单的整合 Security 然后启动时配置账号密码
spring:
    # 配置项，可以在对应的 SecurityProperties 配置类中查看。
  security:
    user:
      name: admin #用户名
      password: 123456 #密码
      roles: 
       - amind # 拥有角色
```

通过 JavaConfig 的方式实现密码登录的配置，如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
// 开启 Security 支持
@EnableWebSecurity 
public class UserLoginSecurityConfig {

    @Bean
    public UserDetailsService userDetailsService() {
        // 直接返回写死内存的用户时，通过 {} 的方式指定密码的加密方式，默认支持的加密类型请查看 PasswordEncoderFactories 类
        UserDetails user = User.withDefaultPasswordEncoder().username("admin").password("{MD5}123456").roles("admin").build();
        // 通过 {noop} 表示不加密密码
        user = User.withUsername("admin").password("{noop}123456").roles("admin").build();
         // 直接写法用户返回，
        return new InMemoryUserDetailsManager(user);
        // 我们也可以通过接口 UserDetailsManager、UserDetailsService 等等接口来，返回用户密码，进行后续的密码校验
    }

    /**
     * 通过该方式，可以构建 UserDetailsService 时，可以不使用 {} 指定加密方式
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        //return NoppPasswordEncoder.getInstance(); //不加密
        //加密方式 bcrypt
        return new BCryptPasswordEncoder();
    }

    /**
     * 自定义过滤器链的，方式改变登录页面（比如我们美化登录页面的时候）
     *
     * @param http
     * @return
     * @throws Exception
     */
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.formLogin(fromLogin ->
                // 一定要和登录页面的登录 api 相同
                fromLogin.loginPage("/login.html").loginProcessingUrl("/user/login").defaultSuccessUrl("/index.html"));
        // 授权处理
        http.authorizeRequests(authorize -> authorize
                // 表示不要授权认证
                .regexMatchers("login.html", "user/login").permitAll()
                // 其他请求都需要授权认证
                .anyRequest().authenticated());
        // 关闭跨越校验
        http.csrf().disable();
        return http.build();
    }

}
```

### 基于注解的访问控制

| 注解 | 作用 | 启用配置 | 特点 |
| :--- | :--- | :--- | :--- |
| `@PreAuthorize` | 方法执行前检查权限 | `prePostEnabled = true` | 最强大，支持SpEL表达式 |
| `@PostAuthorize` | 方法执行后检查权限 | `prePostEnabled = true` | 可基于返回值做检查 |
| `@Secured` | 基于角色的简单检查 | `securedEnabled = true` | 简单直接，不支持SpEL |
| `@RolesAllowed` | JSR-250标准角色检查 | `jsr250Enabled = true` | Java标准，可移植性好 |
| `@PreFilter` | 方法执行前过滤集合参数 | `prePostEnabled = true` | 过滤集合中的元素 |
| `@PostFilter` | 方法执行后过滤集合返回值 | `prePostEnabled = true` | 过滤返回集合中的元素 |

#### 启用权限注解 

```java
@Configuration
// 启用所有类型的权限注解
@EnableGlobalMethodSecurity(
    prePostEnabled = true,   // 启用 @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter
    securedEnabled = true,   // 启用 @Secured
    jsr250Enabled = true     // 启用 @RolesAllowed (JSR-250)
)
public class SecurityConfig {

}
```
#### 注解权限使用例子

第一个例子：

```java
@Service
public class ProductService {
    
    // 1. 基于角色检查，不要求 ROLE_ 前缀
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteProduct(Long productId) {
        // 仅管理员可执行
    }
    
    // 2. 多个角色满足其一即可
    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public void updateProduct(Product product) {
        // 管理员或经理可执行
    }
    
    // 3. 基于权限字符串检查（更细粒度）
    @PreAuthorize("hasAuthority('PRODUCT_DELETE')")
    public void archiveProduct(Long productId) {
        // 需要有 PRODUCT_DELETE 权限
    }
    
    // 4. 组合条件
    @PreAuthorize("hasRole('USER') and hasIpAddress('192.168.1.0/24')")
    public void viewDashboard() {
        // 用户角色且来自特定IP段
    }
    
    // 5. 访问方法参数（强大特性！）
    @PreAuthorize("#userId == authentication.principal.id")
    public UserProfile getUserProfile(Long userId) {
        // 只能查看自己的资料
    }
    
    @PreAuthorize("#product.ownerId == authentication.principal.username")
    public void updateProduct(Product product) {
        // 只能修改自己拥有的产品
    }
    
    // 6. 调用Bean的方法进行权限检查
    @PreAuthorize("@productSecurityService.canView(#productId, authentication)")
    public Product getProduct(Long productId) {
        // 调用自定义安全服务进行复杂检查
    }
    
    // 7. 基于时间的权限控制
    @PreAuthorize("T(java.time.LocalTime).now().isAfter(T(java.time.LocalTime).of(9, 0))")
    public void processBatchJob() {
        // 仅在9点后执行
    }
}
```

第二个例子：

```java
@Service
public class OrderService {
    
    // 1. 基于返回值检查
    @PostAuthorize("returnObject.customerId == authentication.principal.id")
    public Order getOrder(Long orderId) {
        // 方法正常执行，但返回时会检查
        // 只能访问自己的订单
    }
    
    // 2. 结合返回值的属性
    @PostAuthorize("returnObject.status != 'CONFIDENTIAL' or hasRole('ADMIN')")
    public Document getDocument(Long docId) {
        // 非机密文档任何人都可看
        // 机密文档只有管理员可看
    }
    
    // 3. 返回值是集合时的检查（第一个元素）
    @PostAuthorize("returnObject.size() == 0 or returnObject[0].owner == authentication.name")
    public List<Account> findAccountsByType(String type) {
        // 如果没有结果或第一个账户属于当前用户
    }
}
```

第三个例子：

```java
@Service
public class AdminService {
    
    // 1. 单个角色，要求 ROLE_ 前缀
    @Secured("ROLE_ADMIN")
    public void performAdminAction() {
        // 需要 ROLE_ADMIN 角色
    }
    
    // 2. 多个角色（满足其一即可）
    @Secured({"ROLE_ADMIN", "ROLE_SUPER_ADMIN"})
    public void systemConfiguration() {
        // 管理员或超级管理员
    }
}
```

第三个例子：

```java
@Service
public class FinanceService {
    
    // 等价于 @Secured({"ROLE_ACCOUNTANT", "ROLE_AUDITOR"})，注意：不要求 ROLE_ 前缀
    @RolesAllowed({"ACCOUNTANT", "AUDITOR"})
    public FinancialReport generateReport() {
        // 会计师或审计员可访问
    }
}
```

#### 自定义函数注解

```java
@Component("securityUtil")
public class SecurityUtil {
    
    public boolean isOwner(Object obj, Authentication auth) {
        if (obj instanceof OwnableEntity) {
            return ((OwnableEntity) obj).getOwner().equals(auth.getName());
        }
        return false;
    }
    
    public boolean isBusinessHours() {
        LocalTime now = LocalTime.now();
        return now.isAfter(LocalTime.of(9, 0)) && now.isBefore(LocalTime.of(18, 0));
    }
}

// 在表达式中使用自定义函数
@PreAuthorize("@securityUtil.isOwner(#product, authentication) and @securityUtil.isBusinessHours()")
public void updateProduct(Product product) {
    // 产品所有者且在营业时间
}
```


### JWT 认证登录模式

JWT（JSON Web Token ）生成的 Token 携带用户信息，可以通过验证 Token 的合法性，判断是否需要登陆，验证合法的 Token 我们就可以取出登录的用户信息。
注意 Token 支持 RAS 对称性加密，公私钥都可以放在 Nacos 中方便更换。

JWT 一旦签发后就不能撤回了，我们可以设置一个过期时间，当我们校验 Token 的时候，如果时间过期了，就说明 Token 失效了。或者我们可以添加一个黑名单，将 JWT 加入黑名单后就不能访问了。


#### 实现 JWT 登录认证

第一步先实现一个 JWT 检验过滤器

```java

import org.springframework.security.authentication.AuthenticationServiceException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    private static final String TOKEN_PREFIX = "Bearer ";
    private static final String AUTH_NAME = "Authorization";

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String bearerToken = request.getHeader(AUTH_NAME);
        if (bearerToken != null && bearerToken.startsWith(TOKEN_PREFIX)) {
            String authToken = bearerToken.substring(TOKEN_PREFIX.length());
            // todo 校验 token 是有效
            if (authToken != "Hello World") {
                throw new AuthenticationServiceException("Token 是无效的！");
            }
            Authentication authentication = new UsernamePasswordAuthenticationToken(authToken, null, null);
            // 设置到安全上下文
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        // 往下执行其他拦截过滤器
        filterChain.doFilter(request, response);
    }
}

```

第二步将过滤器添加到 Security 拦截过滤器（SecurityFilerChain ）链路中，注意顺序一定要添加在密码登录拦截的过滤器前

```java
import com.vianlex.filter.JwtAuthenticationTokenFilter;
import com.vianlex.handler.LoginFailureHandler;
import com.vianlex.handler.LoginSuccessHandler;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
//开启 Security 支持
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class SecurityConfig {
    /**
     * 自定义过滤器链
     *
     * @param http
     * @return
     * @throws Exception
     */
    //@Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // 授权处理
        http.authorizeRequests(authorize -> authorize
                // 表示不要授权认证
                .regexMatchers("login.html", "user/login").permitAll()
                .regexMatchers("/user/*").hasRole("admin")
                // 其他请求都需要授权认证
                .anyRequest().authenticated());

        // 添加 JWT 过滤器验证，注意要放在 UsernamePasswordAuthenticationFilter 前
        http.addFilterBefore(new JwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```









