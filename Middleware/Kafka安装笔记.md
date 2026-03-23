# Kafka 安装笔记


## Zookeeper 安装 

```bash
# 下载文件（linux 和 window 都是下载相同的文件）
wget https://archive.apache.org/dist/zookeeper/zookeeper‐3.9.4/apache‐zookeeper‐3.9.4‐bin.tar.gz

# 解压下载文件
tar ‐zxvf apache‐zookeeper‐3.5.8‐bin.tar.gz

# 添加默认配置文件
cd apache‐zookeeper‐3.5.8‐bin
cp conf/zoo_sample.cfg conf/zoo.cfg

# linux 启动 zookeeper
bin/zkServer.sh start

# window 启动 zookeeper 
./bin/zkServer.cmd 

```

## Kafka 安装

### Kafka 下载安装

```bash
# 下载文件（linux 和 window 都是下载相同的文件）
wget https://archive.apache.org/dist/kafka/2.8.2/kafka_2.13-2.8.2.tgz

# 解压下载文件
tar ‐zxvf kafka_2.13-2.8.2.tgz

# 进入目录修改 /config/server.properties 后启动即可 
cd kafka_2.13-2.8.2

```

### 修改 Kafka 配置文件

```properties
# server.properties 文件

# 每一个 Broker 的唯一标识符
broker.id=0

# Broker 监听地址
listeners=PLAINTEXT://:9092

# 数据存储目录  
log.dirs=/tmp/kafka-logs

# 网络线程数 
num.network.threads=3

# IO 线程数  
num.io.threads=8  

# 发送缓冲区，异步发送消息时，先发到缓存区，异步后台 IO 线程数，再将消息发送到 Broker
socket.send.buffer.bytes=102400  

# 接收缓冲区
socket.receive.buffer.bytes=102400

# Zookeeper 连接地址，多个 Zookeeper 通过逗号隔开
#zookeeper.connect=localhost:2181,localhost:2182,localhost:2183
zookeeper.connect=localhost:2181

# 定义监听器
listeners=PLAINTEXT://0.0.0.0:9092
# 指定 listeners 定义的监听，哪些是对外网开放的，如不显式指定，则默认开放所有 listeners 定义的监听器
#advertised.listeners=PLAINTEXT://0.0.0.0:9092

# 创建 topic 的默认分区数
num.partitions=1
# 自动创建topic的默认副本数量，建议设置为大于等于2
default.replication.factor=1
# 当 producer 设置 acks 为-1时，min.insync.replicas指定replicas的最小数目（必须确认每一个repica的数据都是成功的），如果这个数目没有达到，producer发送消息会产生异常
min.insync.replicas=1
# 是否允许删除主题
delete.topic.enable=false 

```

### 启动 Kafka 服务

Linux 启动 Kafka 服务

```bash 

# 运行日志在 logs 目录的 server.log 文件里, 参数 ‐daemon 后台启动，不会打印日志到控制台
bin/kafka‐server‐start.sh ‐daemon config/server.properties 

# 或者用通过 & 表示后台启动
bin/kafka‐server‐start.sh config/server.properties &

# 通过 nohup 重定向日志到指定文件，并通过 & 表示后台运行
nohup bin/kafka‐server‐start.sh >> ~/kafka_output.log 2>&1  config/server.properties &

# 停止kafka
bin/kafka‐server‐stop.sh

```

Window 启动 Kafka 服务

```bash

# 启动服务
./bin/windows/kafka-server-start.bat ./config/server.properties

# 停止服务
./bin/windows/kafka-server-stop.bat

```

### Kafka 测试例子

```bash 

# 创建名 test001 的主题
./bin/kafka‐topics.sh ‐‐create ‐‐zookeeper 127.0.0.1:2181 ‐‐replication‐factor 1 ‐‐partitions 1 ‐‐topic test001

 ./bin/windows/kafka-topics.bat  --create  --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 2 --topic test002

# 查看主题信息
 

./bin/windows/kafka-topics.bat --describe --zookeeper 127.0.0.1:2181 --topic test001

## 启动生产者，发送消息
./bin/kafka‐console‐producer.sh ‐‐broker‐list 127.0.0.1:9092 ‐‐topic test001

./bin/windows/kafka-console-producer.bat --broker-list 127.0.0.1:9092 --topic test001 

## 启动消费者，
./bin/kafka‐console‐consumer.sh ‐‐bootstrap-server 127.0.0.1:9092 ‐‐topic test001

./bin/windows/kafka-console-consumer.bat --bootstrap-server 127.0.0.1:9092 --topic test001 

# 查看指定偏移量的消息，注意通过偏移量查看消息，必须指定要查看那个分区的消息
./bin/windows/kafka-console-consumer.bat --bootstrap-server 127.0.0.1:9092 --partition 0 --offset 0  --topic test001

```

## Broker 服务内外网监听器配置说明

listeners 定义监听器，默认只能在内网中 Broker 节点相互访问。如果外网（本地开发工具或者应用）想通过监听器访问 Broker 需要通过 advertised.listeners 属性配置对外外网放开的监听器，外网才能访问 Broker。如果 advertised.listeners 没有显式配置，则默认对外开发所有的 listeners 监听器。

```properties
# 监听器格式：协议://主机名:端口, PLAINTEXT 协议表示无认证无加密，SSL 表示 SSL/TLS 加密

# 定义监听器，集群中 broker 节点的访问链接
listeners=INTERNAL://internal.host.name:9092, EXTERNAL://internal.host.name:9093

# 表示对外公布的监听器，只有把 listeners 定义的监听对外公布后，本地的开发工具或应用才能访问
advertised.listeners=INTERNAL://external.host.name:9092

# 协议别名定义
listener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT

# 指定 broker 间通信使用的监听器名字，默认是 PLAINTEXT
inter.broker.listener.name=INTERNAL

```

注意：`INTERNAL://:9092` 表示监听所有网卡的网络端口，等价于 `INTERNAL://0.0.0.0:9092` 和 `INTERNAL://[::]:9092 (IPv6的写法)`

比如服务器有两块网卡，一个是外网地址网卡，另一个是内网地址网卡，如果我们监听的端口指定 IP 地址为 0.0.0.0，那么通过是内网或者外网网卡地址都可以访问应用。如果我们只单独监听某个网卡的端口，那么只有通过对应网卡的 IP 地址才能访问应用。

