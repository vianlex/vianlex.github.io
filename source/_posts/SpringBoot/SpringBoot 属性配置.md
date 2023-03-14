---
title: SpringBoot 属性配置
---

## 配置读取
SpringBoot 属性配置支持 properties 文件、YAML文件、环境变量和命令行参数。 Spring Boot 提供的 SpringApplication 类会搜索并加载配置文件来获取配置属性值。SpringBoot 启动运行 SpringApplication 类的 run 方法时，会以下位置搜索配置文件：
- 当前目录的 ／config 子目录
- 当前目录
- classpath 中的 /config 包
- classpath


## 配置的访问
1. 通过 @ConfigurationProperties 绑定到 Bean 对象

2. 在 Bean 对象中通过 @Value 注解引用

3. 通过 Spring 提供的 Environment 类访问

