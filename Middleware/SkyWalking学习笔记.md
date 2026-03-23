# SkyWalking 学习笔记


## 安装 SkyWalking 服务 

测试环境直接使用 h2 数据库即可，如下：

```yaml
services:
  # SkyWalking OAP Server
  skywalking-oap:
    image: apache/skywalking-oap-server:10.1.0
    container_name: skywalking-oap
    restart: always
    ports:
      - "11800:11800"   # gRPC API
      - "12800:12800"   # HTTP API
    environment:
      - SW_STORAGE=h2
      - TZ=Asia/Shanghai
    volumes:
      - ./data/logs:/skywalking/logs
    networks:
      - skywalking-network

  # SkyWalking UI
  skywalking-ui:
    image: apache/skywalking-ui:10.1.0
    container_name: skywalking-ui
    depends_on:
      - skywalking-oap
    restart: always
    ports:
      - "18080:8080"
    environment:
      # 注意这里的地址一定要加 http 不然无法访问
      - SW_OAP_ADDRESS=http://skywalking-oap:12800
      - TZ=Asia/Shanghai
    networks:
      - skywalking-network

networks:
  skywalking-network:
    driver: bridge
```

## 配置 Agent 监控应用

第一步：下载 Agent 文件

```bash
wget https://dlcdn.apache.org/skywalking/java-agent/9.5.0/apache-skywalking-java-agent-9.5.0.tgz
```

第二步：应用启动时，配置 agent 探针

比如我们解压 `apache-skywalking-java-agent-9.5.0.tgz` 文件后，将 `skywalking-agent` 文件夹复制到 `D:` 盘下（注意要复制文件夹种的所有文件）。然后我们构建 Java 应用启动参数，如下。

```bash
# 第一种方式
-javaagent:D:\skywalking-agent\skywalking-agent.jar
-Dskywalking.agent.service_name=order-server # 配置要监控的服务，注意：配置的是 spring.application.name 应用名
-Dskywalking.collector.backend_service=localhost:11800 # 配置 skywalking 的服务端地址

# 第二种方式
-javaagent:D:\skywalking-agent\skywalking-agent.jar=agent.service_name=order-server,collector.backend_service=localhost:11800
```

如果我们是通过 idea 启动应用直接将 agent 配置参数，添加到 vm options，然后启动应用即可。

如果我们通过命令启动应用使用如下命令：

```bash
# 基本语法
java -javaagent:/path/to/skywalking-agent.jar \
     -Dskywalking.agent.service_name=your-service \
     -Dskywalking.collector.backend_service=localhost:11800 \
     -Dskywalking.logging.level=INFO \
     -Dskywalking.logging.file_name=skywalking-api.log \
     -Dskywalking.logging.dir=/opt/logs \
     -jar your-application.jar \
     --spring.profiles.active=prod \
     --server.port=8080

# 或使用系统属性格式
java -javaagent:/path/to/skywalking-agent.jar \
     -DSW_AGENT_NAME=your-service \
     -DSW_AGENT_COLLECTOR_BACKEND_SERVICES=localhost:11800 \
     -jar your-application.jar
```

**注意**如果是监控 SpringClod Gateway 网关，Agent 默认是不支持的，我们需要将 `D:\skywalking-agent\optional-plugins\apm-spring-cloud-gateway-4.x-plugin-9.5.0.jar` 文件拷贝到 `D:\skywalking-agent\plugins` 下。同理如果需要监控 `Nacos 客户端`，我们也要复制对应的 jar 插件。



## SkyWalking 集成日志框架

### 引入依赖包

```xml
<!-- SkyWalking 工具类 -->
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>9.3.0</version>
</dependency>

<!-- apm-toolkit-logback-1.x -->
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <version>9.3.0</version>
</dependency>

```

引入日志依赖包后，我们可以在 `logback-spring.xml` 日志配置文件中通过 `%tid` 打印 Trace 链路的 ID。作用是，我们在 Skywalking UI 查看接口调用 Trace 链路时，可以根据 Trace ID 查找应用服务器上与调用链路相关的日志信息。


### 将服务日志上传到 Skywalking 

将微服务的日志信息上传到 Skywalking 中，需要以下配置，这样我们就可以在 Skywalking UI 中查看链路调用对应的日志。

```xml
<!-- 配置 gRPC Appender，将日志文件输入到 GRPCLogClientAppender 中 -->
<appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
    <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
        <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
            <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n</Pattern>
        </layout>
    </encoder>
</appender>

<!-- 指定日志级别 -->
<root level="INFO">
    <appender-ref ref="console"/>
    <appender-ref ref="grpc-log" />
</root>
```

### 自定义日志追踪

