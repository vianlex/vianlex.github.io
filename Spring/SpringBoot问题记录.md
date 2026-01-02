# SpringBoot 问题记录


## Junit 版本不对

运行测试类时 `java.lang.NoSuchMethodError: 'org.junit.jupiter.api.extension.ExecutableInvoker org.junit.jupiter.api.extension.ExtensionContext.getExecutableInvoker()'` 提示找不到方式，这种情况下就是 jar 依赖包版本不对，导致找不到对应的方法。

项目使用的是 `spring-boot-starter-test` 查看 `spring-boot-starter-test` 和 `spring-boot-starter-parent` 的 pom 文件时，发现它们 pom 文件依赖的版本都是 `5.11.4` 但是运行命令 `mvn dependency:tree` 查看当前项目的依赖树，输出的 `junit` 依赖包确是 `5.6.2`，最终排除发现是当前项目 pom 文件的 `properties` 配置有属性 `<junit-jupiter.version>5.6.2</junit-jupiter.version>`，该属性会覆盖 `spring-boot-starter-parent 父级 spring-boot-dependencies ` 配置的 `junit-jupiter.version` 从导致引入版本错误。

解决方式：去掉当前项目配置的 `junit-jupiter.version` 属性即可。





