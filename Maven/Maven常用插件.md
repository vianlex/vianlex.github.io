# Maven 常用插件


## 定义项目编译 JDK 版本插件

比如系统环境安装的是 1.8 版本的 JDK，当我们的项目依赖的 JDK 版本不是1.8时，我们可以通过 `maven-toolchains-plugin` 去指定 maven 编译项目所需的 JDK 版本。具体的使用方式如下：

第一步：先在用户配置目录 `~/.m2/toolchains.xml` 或 `Maven` 安装目录 `conf/toolchains.xml` 创建工具链配置文件。

```xml

<?xml version="1.0" encoding="UTF-8"?>
<toolchains>
    <toolchain>
        <type>jdk</type>
        <provides>
            <!-- JDK 版本 -->
            <version>11</version>
            <!-- JDK 提供商 -->
            <vendor>openjdk</vendor>
        </provides>
        <configuration>
            <!-- JDK 目录 -->
            <jdkHome>/usr/lib/jvm/java-11-openjdk</jdkHome>
        </configuration>
    </toolchain>
    
    <toolchain>
        <type>jdk</type>
        <provides>
            <!-- 相同 JDK 版本，但是由不同提供商提供 -->
            <version>11</version>
            <!-- JDK 提供商 -->
            <vendor>oracle</vendor>
        </provides>
        <configuration>
            <jdkHome>/opt/jdk-11</jdkHome>
        </configuration>
    </toolchain>

    <toolchain>
        <type>jdk</type>
        <provides>
            <version>17</version>
            <vendor>temurin</vendor>
        </provides>
        <configuration>
            <jdkHome>/opt/jdk-17.0.5</jdkHome>
        </configuration>
    </toolchain>
    
</toolchains>

```

第二步：在项目的 `pom.xml` 文件中引入 `maven-toolchains-plugin` 插件，然后配置编译项目时，所依赖的 JDK 版本。

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-toolchains-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
        <execution>
            <goals>
                <goal>toolchain</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <toolchains>
            <jdk>
                <version>11</version>
                <!-- toolchains.xml 文件配置的 jdk 版本只有一个服务商，该配置可以去掉。 -->
                <vendor>openjdk</vendor>
            </jdk>
        </toolchains>
    </configuration>
</plugin>

```