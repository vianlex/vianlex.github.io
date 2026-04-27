# Maven 学习笔记

## Maven 配置优先级

Maven 三类核心配置文件优先级从高到低：

项目级配置（pom.xml） > 用户级配置（用户目录下 settings.xml） > 全局配置（Maven 安装目录下全局 settings.xml）

项目构建时，Maven 会**由高优先级到低优先级**依次加载配置：高优先级配置已定义则直接生效；若缺失、未配置或不满足需求，才会向下读取低优先级配置补全。

注意：仓库、依赖、插件仓库等仓库相关配置，项目 pom.xml 文件中 <repositories> 优先级最高。

## Maven 依赖仓库查找规则

Maven 依赖查找整体流程：本地仓库 → 远程仓库，优先从本地缓存读取，本地缺失则自动从远程仓库拉取并缓存至本地。

### 1. 本地仓库查找逻辑

默认本地仓库路径：`~/.m2/repository` Maven 会优先检索本地仓库是否存在目标 Jar 包；

注意：Maven 会强制校验本地依赖的远程溯源信息，若本地存在依赖但缺失 _remote.repositories 溯源文件，会判定依赖异常、抛出找不到依赖报错。

本地依赖校验报错三种解决方案

方案一（临时生效）：Maven 3.9 之前版本，通过启动参数关闭远程校验，即运行 Maven 构建命令时，加上以下参数：

```bash
# maven 3.9 之前的版本
mvn clean package  -llr 
# 或者
mvn clean package --legacy-local-repository 
# 或者
mvn clean package -Dmaven.legacyLocalRepo=true

# maven 3.9 之后的版本
mvn clean install -Dmaven.artifact.enhancedLocalRepository.enabled=false
```

方案二（手动修复）：删除依赖目录下 _remote.repositories 文件，手动上传本地 Jar 至仓库

```bash
mvn install:install-file -Dfile=D:/xx.jar -DgroupId=com.ils -DartifactId=xx -Dversion=1.0.0 -Dpackaging=jar
```

方案三（永久规避）：新增镜像配置，将镜像地址指向本地仓库目录：`file://D:/maven/repo/`

```xml
<!-- 本地私有镜像：将所有仓库都指向本地仓库 -->
<mirror>
    <id>local-mirror</id>
    <mirrorOf>*</mirrorOf>
    <url>file://D:/maven/repo-local</url>
</mirror>
```


### 2. 远程仓库查找优先级

本地无对应依赖时，自动按以下顺序遍历远程仓库下载：

- 项目 pom.xml 文件中自定义的 <repositories> 仓库
- 用户级 settings.xml 中配置的远程仓库
- 全局 settings.xml 中配置的远程仓库


注意：若同一配置文件内多个仓库，按从上到下顺序依次匹配查找，若目标仓库配置了镜像，会自动转发请求至镜像仓库拉取依赖。


## Maven 镜像仓库


### 1. 镜像核心概念

若 A 仓库完整包含 B 仓库的所有依赖资源，且访问速度更快、稳定性更高，即可将 A 作为 B 的镜像仓库。

访问原远程仓库时，Maven 会通过镜像转发请求，将 B 仓库依赖下载请求转发到 A 仓库，大幅提升构建速度。

### 2. 镜像仓库配置

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

### 3. 镜像仓库 mirrorOf 匹配规则

- `*` 表示匹配所有仓库，即 Maven 通过 repositories 配置的仓库查找依赖时，会将 repositories 仓库的地址替换成 mirror 镜像地址，再去查找依赖包。
- `repo1` 表示只匹配 repo1 仓库
- `repo1,repo2` 表示只匹配 repo1, repo2 两个仓库
- `*,!repo1` 表示匹配除 repo1 外的所有仓库
- `external:*` 表示匹配所有非 file:// 协议的仓库
- `external:http:*`	表示匹配所有 HTTP 协议的仓库
- `external:https:*`	表示匹配所有 HTTPS 协议的仓库


### 4. 多个 mirror 匹配规则说明

4.1 精确匹配优先于通配符

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

4.2 排除语法

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

## release 正式版本和 snapshot 快照版本

Maven 如何判断一个 JAR 包是 `release` 版本还是 `snapshot` 版本，主要是通过版本号后缀 `-SNAPSHOT` 来区分的。

例如：`1.0.0-SNAPSHOT` 版本号带后缀 `-SNAPSHOT` 则为快照版本，`1.0.0` 版本号不带后缀 `-SNAPSHOT` 则为正式版本。

快照版本和正式版本的区别如下：

| 特性 | 快照版本（SNAPSHOT） | 正式版本（Release） |
| ---- | ---- | ---- |
| 命名格式 | 1.0.0-SNAPSHOT | 1.0.0、2.1.5 |
| 使用阶段 | 开发中、未发布 | 已完成、正式发布 |
| 覆盖规则 | 同一版本号可以反复覆盖更新 | 版本号一旦发布，绝对不能修改 / 覆盖 |
| 依赖更新 | Maven 会自动检查最新代码 | 依赖固定不变，必须升级版本号才更新 |

快照版本和正式版本，在仓库存储的区别如下：

1、快照版本（SNAPSHOT）

快照版本在打包构建时，Maven 会自动将版本后缀 -SNAPSHOT 替换为 时间戳 + 编译序号，再缓存到本地仓库或上传至远程仓库；每次新构建都会直接覆盖上一次的构建产物。

注意：快照版本每次构建后，在仓库后的名称都是不一样的，示例如下：

```bash
# 第一次构建
1.0.0-20260427.102233-8.jar
# 第二次构建，会覆盖第一次构建的产物
# 每一次构建都会删除前一次构建的产物，再上传当前构建构建的产物
1.0.0-20260427.125512-9.jar
```

2、正式版本（Release）

严格不可覆盖，如果仓库已存在对应版本的构建产物，重复上传直接报错，正式版本版本必须唯一性。除非手动删除后，才能上传。

正式版本和快照版本最大的区别点就是：构建产物上传仓库时，快照版本号相同能自动实现覆盖更新，正式版本相同只能手动删除更新或者更换版本号


## 依赖仓库配置

```xml
<repository>
    <!-- 仓库ID -->
    <id>aliyun</id>
    <!-- 仓库地址 -->
    <url>https://maven.aliyun.com/repository/public</url>
    <!-- releases 控制仓库是否允许下载【正式版依赖】 -->
    <releases>
        <!-- 是否允许下载 -->
        <enabled>true</enabled>
        <!-- 更新策略
            never：从不检查
            daily：每天检查一次
            always：每次构建都检查
            interval:60：每 60 分钟检查一次
        -->
        <!-- 正式包不会变，从不主动更新，只拉取一次 -->
        <updatePolicy>never</updatePolicy>
        <!-- 校验失败策略
            warn：校验出错只警告（默认）
            fail：校验出错直接构建失败
            ignore：忽略校验
        -->
        <checksumPolicy>warn</checksumPolicy>
    </releases>
    <!-- snapshots 控制仓库是否允许下载【快照版依赖】 -->
    <snapshots>
        <enabled>true</enabled>
        <!-- 快照包是开发中、经常变的，所以要选择每次都要检查快照版本是最新的-->
        <updatePolicy>always</updatePolicy>
    </snapshots>
</repository>
```

## 私有仓库认证

`server` 的 `id` 必须和 `mirror、repository` 的 `id` 一致，`Maven` 才会自动带入账号密码。

```xml
<servers>
    <!-- 公司 Nexus 私服 -->
    <server>
        <id>nexus-releases</id>
        <username>admin</username>
        <password>123456</password>
    </server>
    <server>
        <id>nexus-snapshots</id>
        <username>admin</username>
        <password>123456</password>
    </server>
</servers>
```

## 普通依赖仓库（repository）和插件仓库（pluginRepository）

### 两者区别

- 依赖仓库（repository）：用于管理项目业务依赖的 jar 包（即代码里 import 的那些包）。
- 插件仓库（pluginRepository）：用于管理 Maven 构建插件（clean、compile、test、package、deploy 等底层执行工具）

普通依赖只会去 <repositories> 仓库查找依赖；插件只会去 <pluginRepositories> 仓库查找依赖，互不通用。

| 对比项 | <repository>（repositories 下） | <pluginRepository>（pluginRepositories 下） |
|---|---|---|
| 归属标签 | <repositories> | <pluginRepositories> |
| 作用对象 | 项目业务依赖（如 spring-core、guava、mybatis） | Maven 插件（maven-compiler-plugin、maven-surefire-plugin 等） |
| 用途 | 下载项目运行/编译需要的 jar | 下载构建生命周期用的插件（clean/install/package…） |
| 查找逻辑 | 只在 repositories 列表里找 | 只在 pluginRepositories 列表里找 |
| 配置内容 | id、name、url、releases/snapshots | 和 repository **完全一样**（格式相同、可配 releases/snapshots） |
| 默认值 | 中央仓库（central） | 单独的默认插件仓库（也是 central，但独立配置） |

注意：普通依赖仓库和插件仓库即使仓库 `url` 一样，也必须分别配置；只配 `repositories`，插件依然会去默认插件仓库下载，不会复用 `repositories`。


## 配置示例

1、普通依赖仓库配置

```xml
<repositories>
  <repository>
    <id>aliyun</id>
    <url>https://maven.aliyun.com/repository/public</url>
    <releases><enabled>true</enabled></releases>
    <snapshots><enabled>true</enabled></snapshots>
  </repository>
</repositories>
```
2、插件依赖仓库配置

```xml
<pluginRepositories>
  <pluginRepository>
    <id>aliyun-plugin</id>
    <url>https://maven.aliyun.com/repository/public</url>
    <releases><enabled>true</enabled></releases>
    <snapshots><enabled>false</enabled></snapshots>
  </pluginRepository>
</pluginRepositories>
```

### 为什么要分开

职责隔离不同
- 业务依赖：项目运行时需要。
- 插件：Maven 构建工具本身需要，和业务无关。

更新策略不同
- 插件一般追求稳定、少更新（经常换插件版本容易导致构建异常）。
- 业务依赖迭代快，需要灵活配置 SNAPSHOT/Release。

企业私服仓库权限 / 镜像策略不同
- 依赖仓库：一般情况下，只开放 SNAPSHOT。
- 插件仓库：只允许 Release、禁止 SNAPSHOT。



## 网络代理配置（公司内网 / 翻墙用）

```xml
<proxies>
    <proxy>
        <id>http-proxy</id>
        <active>true</active>
        <protocol>http</protocol>
        <host>127.0.0.1</host>
        <port>7890</port>
        <!-- <username>xxx</username> -->
        <!-- <password>xxx</password> -->
        <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
</proxies>
```

##  多环境(profiles)配置

每个 `profile` 是一套环境（仓库、JDK、属性），通过 `id` 切换。

### 常用 profiles 配置 

```xml
<profiles>
    <!-- 开发环境：JDK8 + 阿里云 -->
    <profile>
        <id>dev-jdk8</id>
        <activation>
            <activeByDefault>true</activeByDefault> <!-- 默认激活 -->
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        </properties>
        <repositories>
            <repository>
                <id>aliyun-public</id>
                <url>https://maven.aliyun.com/repository/public</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>false</enabled>
                </snapshots>
            </repository>
        </repositories>
    </profile>

    <!-- 生产环境：JDK11 + 公司私服 -->
    <profile>
        <id>prod-jdk11</id>
        <properties>
            <maven.compiler.source>11</maven.compiler.source>
            <maven.compiler.target>11</maven.compiler.target>
        </properties>
        <repositories>
            <repository>
                <id>nexus-releases</id>
                <url>http://192.168.1.100:8081/repository/maven-releases/</url>
            </repository>
        </repositories>
    </profile>
</profiles>
```

### 激活指定 profile 配置

```xml
<activeProfiles>
    <activeProfile>dev-jdk8</activeProfile>
</activeProfiles>
```