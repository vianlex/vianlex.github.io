# Maven 学习笔记

## Maven 配置优先级

Maven 配置文件的优先级顺序为：项目配置文件（pom.xml） > 用户配置文件（settings.xml） > 全局配置文件（settings.xml）。
Maven 在构建项目时，会按优先级顺序依次读取种配置文件中的配置，当高优先级文件的配置不存在或者不满足时，会依次查找低优先级的配置文件中是否存在或者满足该配置，如果存在或者满足则读取。

注意: Maven 通过项目配置文件中的 repositories 


## Maven 仓库查找规则

Maven 会检验远程仓库是否存在依赖的 Jar 包，如果存在，才会从本地仓库（~/.m2/repository）查找依赖包，如果本地不存在依赖包，才从远程仓库下载依赖包到本地，

1. 从本地仓库（默认目录 ~/.m2/repos）查找依赖包，注意：从本地仓库查找到依赖包后，会检验远程仓库是否存在该依赖包，不存在报错提示找不到依赖包，有两种解决方式：

- 第一种：3.9 版本之前的 Maven 可通过参数 ` -llr 或者 --legacy-local-repositor 或者 -Dmaven.legacyLocalRepo=true` 关闭该检验
- 第二种：删除依赖包路径种 _remote.repositories 文件然后重新 install 依赖包 `mvn install:install-file -Dfile=D:/xx.jar -DgroupId=com.ils -DartifactId=xx -Dversion=1.0.0 -Dpackaging=jar`
- 第三种：添加一个 mirror 镜像仓库配置，镜像仓库地址设置为本地仓库地址如： file://D:/maven/repo/

2. 本地仓库仓库到依赖包时，从远程仓库查找下载依赖包。从远程仓库查找下载依赖的优先级和顺序规则如下：

- 从项目配置文件中的 repositories 仓库查找依赖包
- 从用户配置文件中的 repositories 仓库查找依赖包
- 从全局配置文件中的 repositories 仓库查找依赖包

注意：如果 repositories 配置有多个仓库，按仓库的配置顺序去依次查找，如果仓库存在镜像仓库则通过镜像仓库查找依赖。


## Maven 镜像仓库

如果在 A 仓库中能查找到 B 仓库中的所有依赖包，我们则可以认为 A 仓库是 B 仓库的镜像仓库。如果 A 仓库的网络访问速度比 B 仓库的网络访问速度快，那么当前我们去 B 仓库查找依赖时，可以通过转发的方式去 A 仓库查找依赖。


1. 镜像仓库的配置说明：

```xml
<!-- 注意存在多个 mirror 时，按 mirror 的先后顺序去匹配 -->
<mirror>
    <id>镜像ID</id>
    <!-- 表示 Maven 通过远程仓库查找依赖包时，会根据远程仓库配置的 ID，来匹配 mirrorOf 中的内容，如果匹配通过，则仓库地址替换成镜像地址，再去查找依赖包。-->
    <mirrorOf>仓库 ID 匹配模式</mirrorOf>
    <name>镜像名称</name>
    <url>镜像地址</url>
</mirror>
```

2. 匹配模式规则说明：

- `*` 表示匹配所有仓库，即 Maven 通过 repositories 配置的仓库查找依赖时，会将 repositories 仓库的地址替换成 mirror 镜像地址，再去查找依赖包。
- `repo1` 表示只匹配 repo1 仓库
- `repo1,repo2` 表示只匹配 repo1, repo2 两个仓库
- `*,!repo1` 表示匹配除 repo1 外的所有仓库
- `external:*` 表示匹配所有非 file:// 协议的仓库
- `external:http:*`	表示匹配所有 HTTP 协议的仓库
- `external:https:*`	表示匹配所有 HTTPS 协议的仓库


3. 多个 mirror 匹配规则说明

3.1 精确匹配优先于通配符

```xml

<mirrors>
    <!-- 对 central 使用这个镜像 -->
    <mirror>
        <id>aliyun-central</id>
        <mirrorOf>central</mirrorOf>
        <url>https://maven.aliyun.com/repository/central</url>
    </mirror>
    
    <!-- 对其他仓库使用这个镜像 -->
    <mirror>
        <id>aliyun-others</id>
        <mirrorOf>*</mirrorOf>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
</mirrors>

```

3.2 排除语法

```xml

<mirrors>
    <!-- 匹配除了 internal-repo 外的所有仓库 -->
    <mirror>
        <id>public-mirror</id>
        <mirrorOf>*,!internal-repo</mirrorOf>
        <url>https://public-mirror.com</url>
    </mirror>
    
    <!-- 为 internal-repo 单独配置 -->
    <mirror>
        <id>internal-mirror</id>
        <mirrorOf>internal-repo</mirrorOf>
        <url>http://internal.company.com</url>
    </mirror>
</mirrors>

```



