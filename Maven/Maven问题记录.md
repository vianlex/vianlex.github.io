# Maven 问题记录

## Maven 依赖 OpenJDK 时，运行 mvn 命令会出现证书验证问题

>报错信息：java.lang.RuntimeException: Unexpected error: java.security.InvalidAlgorithmParameterException: the trustAnchors parameter must be non-empty


1. 第一种临时的解决方式：

```bash
mvn -U clean compile -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true
```

2. 第二种解决方式：

```text
从 Oracle 官网下载下载以下文件
jdk7：下载 javase-jce7.jar http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
jdk8：下载 javase-jce8.jar https://www.oracle.com/java/technologies/javase-jce8-downloads.html

然后将解压后覆盖替换 ${JAVA_HOME}/jre/lib/security 目录对应文件即可

如果还有问题的话，则需要在 ${JAVA_HOME}jre/lib/security/java.security 文件末尾添加以为属性 crypto.policy=unlimited
```

3. 第三种解决方式：

```text
直接更换 OpenJDK 成 OracleJDK。
```

## 本地仓库存在依赖包，但是还是一直提示找不到依赖包（Could not find artifact）

### 原因

Maven 读取本地依赖包时，会有一个 Verifying availability 验证依赖包可用性的过程（执行命令时加上 -e -X 参数，通过日志可看到该过程），如果本地仓库依赖包目录下有 _remote.repositories 文件，则 maven 会不管本地仓库是否存在依赖包，都会校验远程仓库上有对应的依赖才行。

### 解决方式

1. 第一种方式，将本地仓库的依赖包目录下的 __remote.repositories 文件删掉即可；

2. 第二种方式，执行命令时加上 `-llr 或 -Dmaven.artifact.enhancedLocalRepository.enabled=false ` 参数表示不使用 __remote.repositories 文件。


## maven 编译项目时，提示 JDK 版本不对

通过 maven 编译项目时，如果提示以下错误，则说明系统环境中安装的 JDK 版本，与项目中 jar 依赖包支持的 JDK 版本不匹配。

```
jar 中的类文件具有错误的版本 61.0, 应为 52.0，请删除该文件或确保该文件位于正确的类路径子目录中。
```

解决方式：将依赖包的版本进行更换，或者修改系统环境中安装的 JDK 版本。


## maven 编译项目时，找不到对应的 JDK

通过 maven 编译项目时，如果项目出现以下问题，说明 maven-compiler-plugin 找不到指定的 JDK 版本。

```
org.apache.maven.plugins:maven-compiler-plugin:3.13.0:compile (default-compile) on project isrm-backend: Fatal error compiling: 无效的目标发行版: 17 -> [Help 1]
```

解决方式：检查系统环境中是否安装了对应的 JDK 版本。


## maven 项目中子模块的版本，使用属性统一配置时，报错问题

通过属性定义版本号时，maven 规定定义版本的属性名称必须为 revision 或者 sha1 或者 changelist， 否则子模块引用该属性时，会报错。

```xml

<!-- 错误的方式 -->
<version>1.0.0</version>
<project-version>1.0.0</project-version>

<!-- 正确的方式 -->
<revision>1.0.0</revision>
<sha1>1.0.0<sha1>

```

## Maven 仓库手动上传 Jar 项目还是无法下载依赖

### 原因

通过 Maven 私服界面或命令上传第三方 JAR 时，没有勾选 `Generate a POM file with these coordinates` 自动生成 POM 文件。

如果是通过命令行上传的，可以创建一个 POM 文件，然后通过 -DpomFile=<path-to-pom> 指定上传 JAR 包的 POM 文件，简单的 POM 文件示例如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

    <!-- 当前上传 JAR 的坐标信息 -->
    <groupId>com.example</groupId>
    <artifactId>bos-webapi-sdk</artifactId>
    <version>8.2.0</version>

    <!-- 可选：JAR 包的描述信息 -->
    <name>example utils</name>
    <description>一些常用的工具类</description>

    <!-- 可选：当前上传 JAR 包，需要依赖包的列表 -->
    <dependencies>
        <!-- 可选的，如果不指定当前当前上传 JAR 的包的依赖，当我们在使用该 JAR 时，需要而外在项目 POM 文件中，手动添加该 JAR 包的依赖即可 -->
    </dependencies>

</project>
```

通过以下命令，上传第三 JAR 包

```bash
mvn deploy:deploy-file 
  -Dfile=D:/xxx/bos-webapi-sdk-8.2.0.jar 
  -DgroupId=com.kingdee 
  -DartifactId=bos-webapi-sdk 
  -Dversion=8.2.0 
  -Dpackaging=jar 
  -DpomFile=D:/xxx/bos-webapi-sdk-8.2.0.pom 
  -Durl=http://192.168.18.226:8888/repository/maven-releases/ 
  -DrepositoryId=maven-releases
```


### 解决方式

删除之前上传的 JAR 文件，然后重新上传，并且勾选 `Generate a POM file with these coordinates` 。

注意，勾选 `Generate a POM file with these coordinates` 后，Maven 仓库会自动生成的 pom.xml 文件，但是内容非常精简，如下

```xml 
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

    <!-- 当前上传 JAR 的坐标信息 -->
    <groupId>com.example</groupId>
    <artifactId>bos-webapi-sdk</artifactId>
    <version>8.2.0</version>

</project>
```

Maven 仓库自动生成的 POM 文件非常精简，不会包含任何 `<dependencies>` 信息

- 后果：如果你上传的 JAR 本身又依赖了其他库（比如它需要 `commons-lang` 才能运行），这个自动生成的 pom 文件里并没有声明这些依赖。
- 导致的问题：在项目A中依赖这个JAR后，编译时可能会报错，提示找不到 `commons-lang` 中的类。因为你项目A的 Maven 无法得知这个传递性依赖，所以不会自动下载 `commons-lang`。
- 解决办法：你必须到项目A的 `pom.xml` 里，手动添加 `commons-lang` 的依赖。


### 问题解释

`Maven` 仓库存储每一个制品，都有一个严格约定：一个制品由 JAR 文件和 POM 文件共同组成。

- 如果你只上传一个 `JAR` 而不上传对应的 `pom.xml`，在仓库看来，这个“制品”是不完整的。
- 当其他项目尝试通过坐标（`groupId`， `artifactId`, `version`）引用这个 `JAR` 时，`Maven` 会先去仓库找 `.pom` 文件。如果找不到，就会报错，导致构建失败。