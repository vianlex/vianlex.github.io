# Maven 常用命令


## install 命令用法


### 手动安装 jar 到本地仓库

1. 第一步在 [maven 仓库](https://mvnrepository.com/) 找到对应 jar 并手动下载

```xml
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>

```
  
2. 使用 ` mvn install ` 命令将手动下载的 jar 导入本地仓库 

```bash

mvn install:install-file -Dfile=D:/guava-33.0.0-jre.jar -DgroupId=com.google.guava -DartifactId=guava -Dversion=33.0.0-jre -Dpackaging=jar

```

3. 使用 ``` mvn dependency:get ` 命令从指定指定仓库下载 jar 到本地仓库

```bash

mvn dependency:get -DgroupId=com.google.guava  -DartifactId=guava -Dversion=33.3.0-jre 

```

4. 运行命令时使用 -X 参数，会打印运行  maven 命令使用的参数配置

```log

[DEBUG] Message scheme: color
[DEBUG] Message styles: debug info warning error success failure strong mojo project
[DEBUG] Reading global settings from C:\DevTools\apache-maven-3.6.2\bin\..\conf\settings.xml
[DEBUG] Reading user settings from C:\Users\zhouy\.m2\settings.xml
[DEBUG] Reading global toolchains from C:\DevTools\apache-maven-3.6.2\bin\..\conf\toolchains.xml
[DEBUG] Reading user toolchains from C:\Users\zhouy\.m2\toolchains.xml
[DEBUG] Using local repository at C:\Users\zhouy\.m2\repository
[DEBUG] Using manager EnhancedLocalRepositoryManager with priority 10.0 for C:\Users\zhouy\.m2\repository

```




## 参考链接
1. https://maven.org.cn/ref/current/maven-embedder/cli.html
