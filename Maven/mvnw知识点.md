# mvnw 知识点

## 什么是 mvnw 

mvnw 是 Maven Wrapper 的缩写。当我们项目想使用特性的 Maven 版本构建项目时，我们通过 Maven Wrapper 定制当前项目要使用的 Maven 版本。


## 指定项目的 Maven 版本

项目指定使用的 Maven 版本，我只需要在项目的根目录运行以下命令即可。

```bash

# 不指定 maven 版本时，表示项目使用最新的 Maven 版本
mvn wrapper:wrapper 

# 显式指定项目使用的 maven 版本，注意版本要加上引号，不然可能会报错 
mvn wrapper:wrapper -Dmaven="3.8.6" 

```

注意运行 `mvn wrapper:wrapper` 命令后，并未开始下载指定的 Maven 版本，只有我们在项目目录中运行 `./mvnw` 命令后，才会开始下载项目指定使用的 Maven 版本。我们安装 Maven Wrapper 插件时只是在项目中生成了 Maven Wrapper 插件运行时需要的相关文件，如下。

```text
wms-backend
├── .mvn
│   └── wrapper
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
```

另外我们可以将项目目录中的 .mvn 目录和 mvnw、mvnw.cmd 文件都提交到版本库中，这样所有的开发人员都可以使用统一的 Maven 版本来管理项目。


## 更改项目的 maven 版本

当前我们要改变项目使用的 maven 版本时，可以直接运行 `mvn wrapper:wrapper -Dmaven="3.8.6"` 或者修改 maven-wrapper.properties 文件中的 distributionUrl 属性也可以。


## mvnw 命令 

mvnw 命令跟普通 mvn 命令用法以及参数都是相同，我们跟使用普通 mvn 命令一样使用 mvnw 命令来构建项目即可。

注意：当我们运行 `./mvnw` 命令，Maven Wrapper 会去读取 maven-wrapper.properties 文件的 distributionUrl 属性，判断当前项目使用的是什么版本的 Maven , 然后去 ` ~/.m2/wrapper/dists ` 目录该版本是否已经下载，如果未下载，则先下载对应的版本 maven。


mvnw 命令使用示例：

```bash

# 查看当前项目使用的 maven 版本
./mvnw -v 

# 清理和编译项目
./mvnw clean compile

# 编译安装项目到本地 maven 仓库，并跳过测试
./mvnw clean install -DskipTests

```


## `Maven Wrapper`命令常用可选参数说明

- `wrapperVersion` 指定 wrapper 插件版本
- `maven | mavenVersion` 指定项目要使用的 Maven 版本
- `distributionBase` 指定 maven 版本 zip 包的缓存目录，默认是 `~/.m2/wrapper/dists`，我们可以设置为 `-DdistributionBase=.mvw/wrapper/dists `
- `distributionPath` 指定 maven 版本解压后的目录， mvnw 命令构建项目调用该目录的 maven，默认是 `~/.m2/wrapper/dists `
- `distributionUrl` 指定 maven 版本的下载地址，如：`https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.8.6/apache-maven-3.8.6-bin.zip`，不指定下载地址的话，会自动根据命令指定的版本生成。
- `wrapperUrl`	指定 Wrapper JAR 的完整的下载地址，默认自动生成。
- `wrapperBase` 指定本地存储 Wrapper JAR 的目录，默认值 `-DwrapperBase=.mvn/wrapper`


注意：项目安装 `maven wrapper`, 当我们要修改 `maven wrapper` 的参数，直接修改 `maven-wrapper.properties` 配置文件即可。




