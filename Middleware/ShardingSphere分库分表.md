# ShardingSphere 分库分表

## 分库分表概述

### 什么是分库分表

1. 垂直分库：根据业务逻辑来分库，如订单库只存订单相关的表，商品库只存商品相关的表，能提示数据库性能，但是不能解决单表过大的问题。
2. 水平分库：将相同表结构的表，按照一定的规则拆分到不同库中，每个库可以位于不同的服务器，以实现水平拓展。
3. 垂直分表：将字段比较多的表，拆分成多个表，比如将字段较多或者不频繁访问的字段拆分出来，创建一个拓展表，这样可以减小核心表的大小，减少磁盘IO。
4. 水平分表：将大数量的表，按一定的规则拆分成结构完全相同的表，然后每个表都存一部分数据，从达到减少单表的数量两，提高查询效率。

对于实际业务而言，我们比较推崇的做法就是：垂直分库和水平分表。


### 分库分库带来的问题

1. 分库会造成事务一致性问题，需要而外考虑分布式事务问题。

2. 跨节点关联表问题，如果分库分表后，数据分布在不同的服务器上，无法进行联表查询（解决思路：将联表拆分查询，然后再将结果合并）。
   
3. 跨节点多库查询需要考虑分页和排序问题，增加系统复杂性，需要先在每个节点查询排序返回后，再次将不同节点返回的结果进行排序汇总。

4. 公共表联合查询问题，对一些数量表少的不需要进行分库分表，如字典表、参数配置表，如果需要进行表联合查询时，解决方式是每个库都要都存一份（通过定时任务或消息队列保持全局表数据一致，或者强制所有对公共表的操作都同时发送到所有分库执行）。


## 分库分表的核心概念

- 数据节点：数据分片的最小单元，由数据源名称和数据表组成。
- 逻辑表：同一个表结构的所有水平拆分的所有水平表，我们可以总称为一张逻辑表，程序中实体类对象就是逻辑表，通过多个逻辑表可以实现多种分片策略算法。
- 真实表：在分片的数据库中真实存在的物理表。
- 绑定表：分片规则一致的主表和子表。
- 广播表（也称公共表）：在所有的分库中都存在的表，并且它们的表结构和表中的数据在每个数据库中都完全一致，例如字典表。
- 分片键：将数据库(表)进行水平拆分的关键字段，即分库分表是根据那些字段去路由找到对应库和表的。
- 分片算法：通过分片算法将数据进行分片，支持通过 =、BETWEEN 和 IN 分片，算法需要由应用开发者自行实现，可实现的灵活度非常高。
- 分片策略：真正用于进行分片操作的是分片键 + 分片算法，也就是分片策略。


## 常见分片策略

1. 范围切分：如按 ID 的范围拆分，ID 在[1到1千万]的放在第一个库，ID 在[1千万到2千万]放在第二个库，以此类推。该策略的缺点比较明显所有的写都在最后一个库上，读流量也可以能偏移，因为最近增加的数据可能被查询最多。
2. 




## ShardingJdbc 的运行模式

ShardingJdbc 提供了两种运行模式分别是：单机(Standalone)模式、集群(Cluster)模式，默认使用的是单机模式。


### 单机模式

单机模式的配置文件支持持久化、部署简单、运维成本低，但是不支持动态配置更新、无法水平扩展。

单机模式的配置信息，默认是直接配置在 SpringBoot 的配置文件中的，如果我们想配置在 JDBC 数据库中或者配置在其他文件目录中，我们可以做如下配置：

```yaml

spring:
  shardingsphere:
    mode:
      type: Standalone  # 单机模式
      repository:
        type: File  # 直接 JDBC 和 File 文件存储
        props:
          path: /opt/shardingsphere/config.yaml  # 配置文件路径
    overwrite: false #  是否使用本地配置覆盖持久化配置

```

单机模式的配置文件推荐的目录结构如下：

```text

/opt/shardingsphere/
├── config.yaml                   # 主配置文件
├── rules/                        # 规则配置
│   ├── sharding.yaml             # 分片规则
│   ├── readwrite-splitting.yaml  # 读写分离规则
│   └── encrypt.yaml              # 数据加密规则
└── data-sources.yaml             # 数据源配置

```



### 集群模式

集群模式的配置信息存储在中心化注册中心，支持多实例部署，支持配置动态更新，支持治理功能（熔断、禁用等）。集群模式配置说明如下：

```yaml
spring:
  shardingsphere:
    mode:
      type: Cluster  # 集群模式
      repository:
        type: ZooKeeper  # 使用 ZooKeeper 作为注册中心
        props:
          namespace: sharding-sphere-demo  # 命名空间
          server-lists: localhost:2181,localhost:2182  # ZK集群地址
          retryIntervalMilliseconds: 500  # 重试间隔
          maxRetries: 3  # 最大重试次数
          timeToLiveSeconds: 60  # 临时节点存活时间
          operationTimeoutMilliseconds: 5000  # 操作超时时间
```

ShardingSphere 支持 Zookeeper 和 etcd 等注册中心，ShardingSphere 推荐使用 Zookeeper 作为注册中心，需要引入对应的依赖包，如下。

```xml

<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-cluster-mode-repository-zookeeper</artifactId>
    <version>${shardingsphere.version}</version>
</dependency>

```




## ShardingJDBC 数据源问题

项目数据库访问架构

```text

应用层: MyBatis-Plus 或者 MyBatis-Flex 是对 MyBatis 功能的增强
       ↓
持久层: MyBatis (SQL 映射、结果集处理)
       ↓
数据源层: DataSource (标准接口)
       ↓
实现层: ShardingSphere-JDBC | Druid | HikariCP
       ↓
数据库: MySQL | PostgreSQL 等到数据库 

```


ShardingSphere 执行机制

```
// MyBatis 执行查询的调用链：
1. userMapper.selectById(1)  // MyBatis API
   ↓
2. SqlSessionTemplate.selectOne()  // MyBatis 核心
   ↓
3. Executor.query()  // 执行器
   ↓
4. StatementHandler.prepare()  // 语句处理器
   ↓
5. DataSource.getConnection()
   // 如果 DataSource 是通过实例 ShardingSphereDataSource 获取到链接时，则会由 ShardingSphere 链接处理
   // 当我们想代码中想实现多数据源切换时，可以使用 dynimacDataSource 组件，然后通过 dynimacDataSource 提供配置类来将 ShardingSphereDataSource 配置到 dynimacDataSource 管理的数据源中。 
   ↓  
6. ShardingSphereConnection.prepareStatement()  
   // ShardingSphere 在这里进行拦截和改写
   // 根据分片规则，将逻辑SQL改写成实际SQL, 路由到不同的数据库, 查询后合并多个分片的查询结果
   ↓
7. 真实的数据库连接执行SQL

```



## ShardingJdbc 实现分库分表

### 简单使用配置

第一步：使用 ShardingJdbc 分库分表，必须要先引入 ShardingJdbc 的依赖包。

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc</artifactId>
    <version>5.5.2</version>
</dependency>
```

第二步：通过配置文件配置 ShardingJdbc 的数据源和分库分表规则

```yaml
spring:
  # 分表分库配置
  shardingsphere:
    mode:
      type: Standalone
    props:
      sql-show: true
    datasource:
      names: srm0, srm1
      srm0:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/srm0?useSSL=false&serverTimezone=UTC
        username: root
        password: 123456
      srm1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/srm1?useSSL=false&serverTimezone=UTC
        username: root
        password: 123456
    rules:
      sharding:
        default-database-strategy:
          standard:
            sharding-column: id
            # 定义分库使用的分片算法
            sharding-algorithm-name: database-mod
        tables:
          user:
            # 表示 user 在哪些库中，分成了多少表，shardingJdbc 会根据我们的分配算法，找到实际进行数据库操作。
            actual-data-nodes: srm$->{0..1}.user_$->{1..2}
            table-strategy:
              standard:
                sharding-column: id
                sharding-algorithm-name: user-inline
            # 通过 shardingJdbc 生成 ID，注意要去掉 Mybatis-plus 的 ID 注解，sharding 生成的 ID 会覆盖 Mybatis-plus 生成的 ID 多余生成了。
            key-generate-strategy:
              # 指定主键字段
              column: id
              # 指定生成主键的算法
              key-generator-name: snowflake
        # 配置分库分表的算法规则，算法配置说明：https://shardingsphere.apache.org/document/current/cn/user-manual/common-config/builtin-algorithm/sharding/
        sharding-algorithms:
          user-inline:
            # 使用 shardingJdbc 内置的标准算法，是一种内行表达式算法，默认实现 Groovy 的表达式
            type: INLINE
            props:
              algorithm-expression: user_$->{id % 2 + 1}
          database-mod:
            # 使用 shardingJdbc 内置的取模算法
            type: MOD
            props:
              # 测试库，只有 2 个库，对 2 取模即可
              sharding-count: 2
        # 配置生成 key 的算法规则
        key-generators:
          snowflake:
            type: SNOWFLAKE
```


## 分片策略和分片算法


### 分片策略

ShardingJdbc 支持四种分片策略分别是：standard、complex、hint、none。

- standard 表示单一分片键，分片字段只支持设置一个分片字段
- complex 表示多分片键，支持设置多个分片字段
- hint  表示强制路由分片策略，不通过 SQL 解析路由分片，而是在代码强制写死要使用的分片
- none 表示不进行分片的表，一般用于广播表，所有数据节点都存在，适用于字典表、配置表等小表。


以 dict 表为例，广播表的使用配置如下，广播表配置好后，当我们插入数据时 `ShardingJdbc` 会将数据写到全库的 dict 表里面，当我们查询时，`ShardingJdbc` 会根据内置算法随机从一个库 dict 表的数据。

```yaml
spring:
   shardingsphere:
    rules:
      sharding:
         tables:
            dict:
            actual-data-nodes: srm$->{0..1}.dict
            table-strategy: none
      broadcast-tables: dict
```

### 分片算法


1. standard 分片策略自定义算法配置

```yaml

spring:
   shardingsphere:
    rules:
      sharding:
        tables:
         order:
            actual-data-nodes: srm$->{0..1}.order_$->{1..2}
            table-strategy:
              standard:
                sharding-columns: id
                sharding-algorithm-name: order-table-alg
        sharding-algorithms:
         order-table-alg:
            # 自定义分表路由算法
            type: CLASS_BASED
            props:
              strategy: standard
              # 按照 = 或者 IN 逻辑的精确分片的自定义算法，需要实现 PreciseShardingAlgorithm 接口
              preciseAlgorithmClassName: com.vianlex.algorithm.CustomPreciseShardingAlgorithm
              # 按照 Between 条件进行的范围分片的自定义算法，需要实现 RangeShardingAlgorithm 接口
              rangeAlgorithmClassName: com.vianlex.algorithm.CustomRangeShardingAlgorithm

```


2. complex 分片策略自定义算法配置

```yaml
spring:
   shardingsphere:
    rules:
      sharding:
        tables:
          t_order:
            actual-data-nodes: srm$->{0..1}.t_order_$->{1..2}
            table-strategy:
              complex:
                sharding-columns: id, user_id
                sharding-algorithm-name: order-complex
        sharding-algorithms:
         order-complex:
            # 自定义分表路由算法
            type: CLASS_BASED
            props:
              strategy: complex
              # 需要实现 ComplexKeysShardingAlgorithm 类
              algorithmClassName: com.vianlex.algorithm.CustomComplexAlgorithm
 
```

3. hint 分片策略自定义算法

```yaml
spring:
   shardingsphere:
    rules:
      sharding:
         tables:
            table-strategy:
               hint:
                  sharding-algorithm-name: user_table_alg
         sharding-algorithms:
            user_table_alg:
               # 自定义分表路由算法
               type: CLASS_BASED
               props:
               strategy: hint
               # 需要实现 HintShardingAlgorithm 类
               algorithmClassName: com.vianlex.algorithm.CustomHintShardingAlgorithm
```

注意： 一个同结构的分表，如果想要定义多种分片算法，我们可以通过定制多个逻辑表，来实现多少分片策略算法。


#### 分片算法问题

1. 如果 `UPDATE` 或者 `SELECT SQL` 语句中的 `WHERE` 条件中没有分片字段时，`ShardingJdbc` 会查询或者更新全部所有分库中的所有分表。 

2. 当分片键通过 IN 作为 WHRER 条件时，如果 `ShardingJdbc` 根据分片算法，如果不能确定 IN 的值都在同一个分片中，则默认会进行全分片查询，即查询所有库中的所有表。 



## 常用分片方案策略

1. 单分片键取模分片，优点是：数据存放比较均匀，缺点：扩容需要大量的迁移数量

2. 单分片键按范围分片，优点：扩容不需要迁移数据，缺点：数据会存在偏移，如按日期分片，可能某个日期范围内的数据少，其他日期范围内的数据多，写操作也会存在偏移，同样也是按日期分片时，写入时都是写入最大日期的分片表中，读操作也同样会存在偏移，因为我们一般读比较多的都是最近写入的数据。

3. 多分片键先按范围分片，然后取模分片。例如，我们可以将 BD 分成多个组，如 [{grou1: bd1 ~ bd3} , {group2: db4~ db8}], 以订单表为例，我们可以先按订单月份分片来路由到组，然后再通过订单 ID 取模分片路由存放到具体的库。比如我们对订单 hash(orderId) % 10 后，可以将取模的值按 [0, 2, 4] 路由到 db1 库，[1, 3, 5] 路由到 db2 库，以此类推。

4. 基因分片，基因分片算法是将两个分片键的后几位数合并在一起，比如我们取用户名 username 哈希值的二进制后3位（基因片段），拼接到用户 ID 后面，形成新的 ID, 那么我们只要对 8 取模分片，那么无论是对应新用户 ID 取模，还是用用户名哈希值取模得到的值都是一样的。从而解决用户表按用户 ID 分表，按用户名查询时，也能路由到具体的表的问题，对于其他表要想达到此相关的我们也可以使用基因分片。

基因分片原理

```java
// 十进制取模，对10取模时，取模结果只和最后 1 位数字有关，如：
System.out.println(19 % 10 == 119 % 10 ); // 输出 true
System.out.println(19 % 10 == 1219 % 10); // 输出 true

// 十进制取模，对100(10^2)取模时，取模结果只和最后 2 位数字有关，如：
System.out.println(199 % 100 == 11199 % 100 ); // 输出 true
System.out.println(199 % 100 == 1121199 % 100); // 输出 true

// 同理，对一个二进制的数字，按 2^n 取模，也只与这个数字的最后 n 位有关，如：
0b10000111 % 0b1000 = 0b111 // 对 2^3 取模输出 7
0b10011100111 % 0b1000 = 0b111 // 对 2^3 取模输出 7

// 只要后 3 位相同，那么取模的结果就是相同的
0b10000101 % 0b1000 = 0b111 // 对 2^3 取模输出 5
0b10011100101 % 0b1000 = 0b111 // 对 2^3 取模输出 5

```

用户名和用户 ID 基因分片，例子如下：

```java
String username = "vianlex";
// 计算 username 的 hashCode
int usernameHashCode = username.hashCode();
// 通过位与运算取 hashCode 的后三位
int mask = 0b111;
// username 的基因片段
int usernameGenes = usernameHashCode & mask;
// 将 oldId 左移动三位
Long oldId = 1871219912101L;
long newId = (oldId << 3) | usernameGenes;
// 输出 true
System.out.println(usernameHashCode%8 == newId%8);

```




