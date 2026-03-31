# Go 依赖管理

## 一、GOPATH 时代（Go 1.0 ~ 1.10）

### 什么是 GOPATH？

GOPATH 是 Go 早期的工作区机制，所有 Go 代码必须放在 GOPATH 指定的目录下。

查看当前 GOPATH
```bash
go env GOPATH
```
默认值：~/go（Linux/macOS）或 %USERPROFILE%\go（Windows）

### 目录结构

```
$GOPATH/
├── src/          # 源代码（你的项目 + 依赖）
│   ├── github.com/user/myproject/
│   └── github.com/gin-gonic/gin/
├── pkg/          # 编译缓存（.a 文件）
│   └── linux_amd64/
└── bin/          # 编译后的可执行文件
```

### 工作方式

下载依赖（放到 $GOPATH/src）
```bash
go get github.com/gin-gonic/gin
```

代码中导入
```bash
import "github.com/gin-gonic/gin"
```

### GOPATH 的问题

- 无版本管理  go get 永远拉最新代码，无法锁定版本
- 全局共享  所有项目共用同一份依赖，版本冲突无解
- 必须在 GOPATH 下  项目路径受限，不灵活
- 无法复现构建  不同机器/时间构建结果可能不同

## 二、Vendor 机制（Go 1.5+）

### 什么是 Vendor？

Vendor 是将依赖代码直接复制到项目目录的方案，解决了依赖版本不一致的问题。

### 目录结构

```txt
myproject/
├── main.go
├── vendor/              # 依赖代码全部放这里
│   ├── github.com/
│   │   └── gin-gonic/
│   │       └── gin/
│   └── vendor.json      # 依赖清单（工具生成）
└── ...
```

### 工作方式

使用 dep 工具（当时最流行的 vendor 管理工具）

```bash
dep init          # 初始化
dep ensure        # 安装/更新依赖到 vendor/
dep ensure -add github.com/gin-gonic/gin  # 添加依赖
```

编译时自动优先使用 vendor/ 目录
```bash
go build ./...
```

### Vendor 的优缺点

优点 ✅
- 依赖代码随项目一起提交，构建完全可复现
- 离线可构建
- 版本锁定

缺点 ❌
- vendor 目录体积庞大，污染 git 仓库
- 工具不统一（dep、glide、govendor 各自为战）
- 官方没有标准方案，生态混乱

## 三、Go Modules（Go 1.11+ 官方标准）

### 什么是 Go Modules？

Go 1.11 引入、1.16 默认开启的官方依赖管理方案，彻底解决了 GOPATH 和 Vendor 的问题。

## 核心文件

go.mod — 依赖声明文件
```
module github.com/myuser/myproject  // 模块路径

go 1.21  // Go 版本要求

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/stretchr/testify v1.8.4
)

// 间接依赖（被依赖的依赖）
require (
    golang.org/x/net v0.17.0 // indirect
)

// 替换依赖（本地开发/镜像）
replace github.com/some/pkg => ../local-pkg

// 排除某个版本
exclude github.com/bad/pkg v1.2.3
```

go.sum — 依赖校验文件

```bash
# 每个依赖有两行：模块 hash + go.mod hash，防止依赖被篡改
github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqJ+Jmg=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL7YrpYKXt5YId3A/Tnip5kqbEAP+KLuI3SUcPTeU=
```

### 常用命令

```bash
#初始化模块
go mod init github.com/myuser/myproject

#添加/更新依赖, 
go get github.com/gin-gonic/gin          # 最新版
go get github.com/gin-gonic/gin@v1.9.1   # 指定版本
go get github.com/gin-gonic/gin@latest   # 最新稳定版

#整理依赖（删除未用的，补充缺失的）
go mod tidy

#下载所有依赖到本地缓存
go mod download

#查看依赖图
go mod graph

#验证依赖完整性
go mod verify

#将依赖复制到 vendor/（可选，用于离线构建）
go mod vendor

#查看可用版本
go list -m -versions github.com/gin-gonic/gin
```

### 版本选择规则（MVS）

Go Modules 使用 最小版本选择（Minimum Version Selection） 算法：

- 项目 A 依赖 gin v1.8.0
- 项目 A 依赖 B，B 依赖 gin v1.9.1

→ 最终选择 gin v1.9.1（满足所有需求的最小版本），不会自动升级到更高版本，保证构建稳定性

语义化版本
```
v1.9.1
│ │ └── Patch：bug 修复，向后兼容
│ └──── Minor：新功能，向后兼容
└────── Major：破坏性变更
```

v2+ 需要修改模块路径！
```
import "github.com/gin-gonic/gin/v2"
```

### 本地缓存位置

- 所有下载的模块缓存在：`$GOPATH/pkg/mod/`

- 查看缓存：`ls $GOPATH/pkg/mod/cache/download/`

- 清理缓存：`go clean -modcache`

## 四、三者对比总结

|特性 | GOPATH |  Vendor | Go Modules
|:--| :--| :--| :--|
|版本锁定 | ❌ | ✅ | ✅|
|官方支持 | ✅  |半官方  ✅ | 官方标准
|项目位置限制 |  必须在 GOPATH | 无限制  | 无限制 |
|离线构建 | ❌ | ✅ | ✅（需缓存）|
|git 仓库体积  | 小 |   大 | 小 |
|构建可复现 |  ❌ | ✅ | ✅ |
|当前状态 |  已淘汰 |  可选辅助 |  推荐使用|

## 五、Go Modules 实战配置

### 国内加速（必备）

设置代理（七牛云，推荐）
- 使用七牛云： `go env -w GOPROXY=https://goproxy.cn,direct`
- 使用阿里云: `go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct`

私有仓库不走代理
```
go env -w GOPRIVATE=gitlab.mycompany.com
```

关闭校验（私有仓库）
```
go env -w GONOSUMCHECK=gitlab.mycompany.com
```

### 典型项目结构

```bash
myproject/
├── go.mod          # 依赖声明
├── go.sum          # 依赖校验
├── main.go
├── internal/       # 内部包（外部不可导入）
├── pkg/            # 可复用包
├── cmd/            # 多个可执行程序入口
│   ├── server/
│   └── cli/
└── vendor/         # 可选，go mod vendor 生成
```

### 常见问题

问题1：依赖下载失败

解决：设置 GOPROXY, 如下设置七牛云加速代理：`go env -w GOPROXY=https://goproxy.cn,direct`

问题2：go.sum 校验失败
```bash
go mod verify
go clean -modcache && go mod download
```

问题3：想用本地修改的依赖
```bash
# 在 go.mod 中添加：
replace github.com/some/pkg => ../local-fork
```

问题4：升级所有依赖
```bash
go get -u ./...
go mod tidy
```

## 六、演进时间线

- 2009  Go 发布，GOPATH 机制
- 2015  Go 1.5，引入 Vendor 支持
- 2016  dep 工具发布（社区标准）
- 2018  Go 1.11，Go Modules 实验性引入
- 2019  Go 1.13，GOPROXY 默认开启
- 2021  Go 1.16，Go Modules 默认开启，GOPATH 模式退出历史舞台
- 2022  Go 1.17+，go.mod 更精确记录间接依赖
