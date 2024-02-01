# Maven 常用命令


## install 命令用法


### 手动安装 jar 到本地仓库

1. 第一步在 [maven 仓库](https://mvnrepository.com/) 找到对应 jar 并下载

```xml
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>

```

2. 使用 ` mvn install ` 命令将 jar 导入本地仓库 

```bash

mvn install:install-file -Dfile=D:/guava-33.0.0-jre.jar -DgroupId=com.google.guava -DartifactId=guava -Dversion=33.0.0-jre -Dpackaging=jar

```

