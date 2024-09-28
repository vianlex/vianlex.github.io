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

### 本地仓库存在依赖包，但是还是一直提示找不到依赖包（Could not find artifact）

#### 原因

Maven 读取本地依赖包时，会有一个 Verifying availability 验证依赖包可用性的过程（执行命令时加上 -e -X 参数，通过日志可看到该过程），如果本地仓库依赖包目录下有 _remote.repositories 文件，则 maven 会不管本地仓库是否存在依赖包，都会校验远程仓库上有对应的依赖才行。

#### 解决方式

1. 第一种方式，将本地仓库的依赖包目录下的 __remote.repositories 文件删掉即可；

2. 第二种方式，执行命令时加上 `-llr 或 --legacy-local-repository ` 参数表示不使用 __remote.repositories 文件；

3. 第三种方式，执行命令时，指定命令以 ` --offline 或 -o ` 离线模式运行，该解决方式没有作用，因为 Maven 是强依赖于远程仓库的。
