# Maven 仓库类型知识点

## Maven 仓库类型

Maven 的核心仓库类型有三种分别为：Hosted(宿主仓库)、Proxy(代理仓库)、Group(组仓库)。

### Hosted(宿主仓库)

Hosted(宿主仓库)主要是用来存放自己上传的私有构件，比如公司内部开发的 SDK、等其他第三方 SDK 依赖、或者自己发布的正式版 / 快照版。

1、常见的 Hosted 仓库：

maven-releases：存放正式发布版（Release）

maven-snapshots：存放开发快照版（Snapshot）

2、Hosted 仓库的特点：

- 支持上传、部署（mvn deploy）
- 可设置部署策略（Deployment Policy）
    - Allow redeploy：允许覆盖（一般给 snapshots 用）
    - Disable redeploy：禁止覆盖（一般给 releases 用，保证版本不可变）
    - Read-only：只读，不能上传 / 修改


## Proxy（代理仓库）

Proxy（代理仓库） 主要用来代理外部公共仓库，比如 Maven Central、阿里云 Maven、JCenter 等。

当 Maven 从这里下载依赖时，Nexus 会先去远程拉取，缓存到本地，再返回给你。

### 常见的 Proxy 仓库：
- maven-central：代理 Maven 中央仓库
- aliyun-maven：代理阿里云 Maven 镜像

### Proxy 仓库特点：

- 不支持上传 / 部署，只能下载
- 第一次请求会去远程拉取，之后直接从 Nexus 缓存返回
- 可设置缓存时间、远程地址、是否允许 SNAPSHOT 等
- 能大幅提升内网下载速度，减少公网依赖请求

### Proxy 配置说明

1. 登录进入 Nexus 
2. 点击 setting → Repositories
3. 选择 Proxy 仓库（默认为 maven-central），
4. 找到 Remote Storage 配置项，添加 `https://maven.aliyun.com/repository/public/` 代理仓库保存即可。
5. 如果想代理多个仓库，我们手动创建新的 Proxy 代理仓库，然后重复上述的步骤配置即可。

## Group（组仓库）

Group（组仓库）主要用途是把多个仓库（Hosted + Proxy）组合成一个地址，对外统一提供服务。

比如你常用的 maven-public 就是一个 Group 仓库。

### Proxy 仓库特点

- 本身不存储任何构件，只是一个 “聚合入口”
- 会按你配置的顺序，依次去成员仓库里找依赖：
    1. 先找 Hosted 仓库（maven-releases / maven-snapshots）
    2. 再找 Proxy 仓库（maven-central / 阿里云）
- 对外只需要配置一个仓库地址，不用分别配置多个
- 成员仓库的顺序会影响查找优先级（靠前的先被访问）

### Group 配置说明

1. 登录 Nexus → 点击 Repositories
2. 找到 maven-public 点击它
3. 找到 Group → Members 配置区
4. 将 Available 可用仓库 → 添加到 Members 仓库
5. 注意仓库的顺序为：maven-releases、maven-snapshots、maven-central