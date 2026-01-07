# Nacos 学习笔记


## SpringBoot 使用 Nacos 配置

### 启动读取配置

SpringBoot 启动读取 Nacos 配置时，遵循以下两种路径。

#### 路径①：直接模式（显式指定 data-id）

当在 `bootstrap.yml 或者 application.yml` 中明确配置了 `spring.cloud.nacos.config.data-id` 时：

完全跳过自动推导逻辑

直接使用指定的 data-id 值

注意：此时 `spring.application.name` 和 `spring.profiles.active` 对于配置加载完全无效

例如：指定 `data-id: global-config.yaml`，无论应用名和环境是什么，都只加载这个固定配置

#### 路径②：推导模式（默认行为）

使用推到模式，在 Nacos 配置中心，`dataId` 的标准命名格式为 `${prefix}-${profile}.${file-extension}`。其中：

- `prefix` 默认来源于项目本地配置文件中 `spring.application.name` 的属性值；

- `profile` 对应项目本地配置中的环境标识，通过 `spring.profiles.active` 指定（例如 `dev`、`prod` 等）；

- `file-extension` 表示配置文件的扩展名，例如 `yaml` 或 `properties`。

通过以上的命名规范有助于 `SpringBoot` 自动识别并加载对应环境的配置。


1. 当没有显式指定 data-id 时，进入自动推导流程：

   - 读取三个关键配置：
   - spring.application.name (如：user-service)
   - spring.profiles.active (如：dev)
   - file-extension (如：yaml)
  
2. 按照从具体到通用的原则生成候选列表

##### 推导模式下，Nacos 客户端会按以下顺序尝试查找（以 user-service、dev 环境、yaml 格式为例）


推导模式下，Nacos 客户端会按以下顺序尝试查找（以 user-service、dev 环境、yaml 格式为例）：


| 优先级        | DataId 模板                           | 实际示例                | 说明                                 |
| ------------- | ------------------------------------- | ----------------------- | ------------------------------------ |
| **1（最高）** | `${prefix}-${profile}.${extension}`   | `user-service-dev.yaml` | **最具体**，完全匹配应用和环境       |
| **2**         | `${prefix}.${extension}`              | `user-service.yaml`     | **通用配置**，所有环境共用           |
| **3**         | `${prefix}`                           | `user-service`          | 无后缀格式（历史兼容）               |
| **4**         | `application-${profile}.${extension}` | `application-dev.yaml`  | 应用名默认为`application` 的环境配置 |
| **5（最低）** | `application.${extension}`            | `application.yaml`      | 默认全局配置                         |


查找规则：

- 按优先级从高到低依次尝试
- 找到第一个存在的配置即停止查找
- 配置合并：如果找到了 user-service-dev.yaml，系统仍然会合并 user-service.yaml 的配置（如果存在），环境特定配置优先级更高


### 配置加载与合并

```text

最低 ← 本地 bootstrap 或者 application 配置
      ← dataId 为 ${prefix}.${extension} 的配置 (如 user-service.yaml)
      ← dataId 为 ${prefix}-${profile}.${extension} 的配置 (user-service-dev.yaml) [覆盖通用]
      ← shared-configs 配置 (多个 shared-configs 按配置顺序加载，后加载的优先级高)
      ← extension-configs 配置 (多个 extension-configs，按配置顺序加载，后加载的优先级高)
      ← 启动参数 (命令行参数) → 最高

```


