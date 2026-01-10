# Nacos 学习笔记


## SpringBoot 整合 Seata 配置

### 不使用注册中心配置

```yaml
seata:
  enable: true
  application-id: ${spring.application.name}
  # 定义微服务所属的分布式事务组，注意 tx-service-group 配置的值 service.vgroup_mapping 配置的属性
  tx-service-group: my_tx_group
  # 自动代理数据源
  enable-auto-data-source-proxy: true  
  service:
    vgroup-mapping:
      # 表示同一个 my_tx_group 微服务事务分组的服务（注意：要将使用同个 Seata TC 服务的，微服务都定义为相同的事务组），使用 default 集群的 Seata TC 服务 
      my_tx_group: default
    grouplist:
      # 设置 Seata Server 的服务地址
      default: 127.0.0.1:8091  
```

### 使用 Nacos 作为配置中心配置

```yaml

# Seata 配置
seata:
  enabled: true
  application-id: ${spring.application.name}
  # 事务组名称
  tx-service-group: my_tx_group  
  # 自动代理数据源
  enable-auto-data-source-proxy: true  
  config:
    type: nacos
    nacos:
      server-addr: vianlex-pc:8848
      # 注意 namespace 填的是命名空间的 ID 值
      namespace: seata-server
      group: SEATA_GROUP
      # 服务和客户端的配置，可以写在同一份 seataSrver.properties 文件，方便维护。
      data-id: seataSrver.properties
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: vianlex-pc:8848
      namespace: seata-server
      group: SEATA_GROUP
      # 定义客户端注册到 nacos 的集群名称
      cluster: default
  service:
    vgroup-mapping:
      # 事务组映射到 Seata 集群，表示事务组使用的是哪个集群的 seata-server 服务端
      my_tx_group: default

```