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