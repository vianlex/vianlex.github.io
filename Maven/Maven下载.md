# Maven 下载 Jar


## 手动下载并并导入仓库

```xml

<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>

```

1. 第一步在 [maven 仓库](https://mvnrepository.com/) 找到对应 jar 并下载

2. 使用 ` mvn install ` 命令将 jar 导入本地仓库 

```
mvn install:install-file -Dfile=D:/guava-33.0.0-jre.jar -DgroupId=com.google.guava -DartifactId=guava -Dversion=33.0.0-jre -Dpackaging=jar

```

