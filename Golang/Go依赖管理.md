# Go 依赖管理

## 第一章 演进历史

### 1.1 三代依赖管理方案

| 时代 | 方案 | Go 版本 | 状态 |
|------|------|---------|------|
| 第一代 | GOPATH | 1.0 ~ 1.10 | 已淘汰 |
| 第二代 | Vendor | 1.5 ~ 1.16 | 可选辅助 |
| 第三代 | Go Modules | 1.11+ | 官方标准 |

### 1.2 演进时间线

```
2009年  Go 发布，GOPATH 机制诞生
2015年  Go 1.5，引入 Vendor 支持
2016年  dep 工具发布（社区标准）
2018年  Go 1.11，Go Modules 实验性引入
2019年  Go 1.13，GOPROXY 默认开启
2021年  Go 1.16，Go Modules 默认开启
2022年  Go 1.17+，go.mod 更精确记录间接依赖
```

## 第二章 GOPATH 机制

### 2.1 什么是 GOPATH

GOPATH 是 Go 早起的工作区机制，所有代码必须放在 GOPATH 目录下。

```bash
go env GOPATH
# 默认值：~/go（Linux/macOS）或 %USERPROFILE%\go（Windows）
```

### 2.2 目录结构

```
$GOPATH/
├── src/          # 源代码（项目 + 依赖）
│   ├── github.com/user/myproject/
│   └── github.com/gin-gonic/gin/
├── pkg/          # 编译缓存（.a 文件）
│   └── linux_amd64/
└── bin/          # 编译产物
```

### 2.3 工作方式

```bash
# 下载依赖到 $GOPATH/src
go get github.com/gin-gonic/gin

# 代码中导入
import "github.com/gin-gonic/gin"
```

### 2.4 存在的问题

- **无版本管理**：`go get` 永远拉最新代码，无法锁定版本
- **全局共享**：所有项目共用同一份依赖，版本冲突无解
- **路径受限**：必须在 GOPATH 下，项目位置不灵活
- **构建不可复现**：不同机器/时间构建结果可能不同

## 第三章 Vendor 机制

### 3.1 什么是 Vendor

Vendor 是将依赖代码直接复制到项目目录的方案。

### 3.2 目录结构

```
myproject/
├── main.go
├── vendor/
│   └── github.com/
│       └── gin-gonic/
│           └── gin/
└── vendor.json
```

### 3.3 工作方式

```bash
# 使用 dep 工具管理
dep init
dep ensure
dep ensure -add github.com/gin-gonic/gin

# 编译时自动优先使用 vendor/
go build ./...
```

### 3.4 优缺点

**优点**
- 依赖代码随项目提交，构建完全可复现
- 离线可构建
- 版本锁定

**缺点**
- vendor 目录体积庞大，污染 Git 仓库
- 工具不统一（dep、glide、govendor 各自为战）
- 官方无标准方案

## 第四章 Go Modules（推荐）

### 4.1 核心文件

#### go.mod — 依赖声明文件

```go
module github.com/myuser/myproject

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/stretchr/testify v1.8.4
)

require (
    golang.org/x/net v0.17.0 // indirect
)

replace github.com/some/pkg => ../local-pkg

exclude github.com/bad/pkg v1.2.3
```

#### go.sum — 依赖校验文件

```bash
# 格式：模块名 版本 h1:Hash / go.mod Hash
github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqJ+Jmg=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL7YrpYKXt5YId3A/Tnip5kqbEAP+KLuI3SUcPTeU=
```

### 4.2 常用命令

```bash
# 初始化模块
go mod init github.com/myuser/myproject

# 添加依赖
go get github.com/gin-gonic/gin          # 最新版
go get github.com/gin-gonic/gin@v1.9.1   # 指定版本
go get github.com/gin-gonic/gin@latest   # 最新稳定版

# 整理依赖（删除未用的，补充缺失的）
go mod tidy

# 下载所有依赖
go mod download

# 查看依赖图
go mod graph

# 验证依赖完整性
go mod verify

# 复制依赖到 vendor/（离线构建）
go mod vendor

# 查看可用版本
go list -m -versions github.com/gin-gonic/gin
```

### 4.3 版本选择规则（MVS）

Go Modules 使用**最小版本选择（Minimum Version Selection）**算法：

```
项目 A 依赖 gin v1.8.0
项目 A 依赖 B，B 依赖 gin v1.9.1

→ 最终选择 gin v1.9.1（满足所有需求的最小版本）
```

**语义化版本**

```
v1.9.1
 │ │ └── Patch：bug 修复，向后兼容
 │ └──── Minor：新功能，向后兼容
 └────── Major：破坏性变更
```

**v2+ 必须修改模块路径**

```go
import "github.com/gin-gonic/gin/v2"
```

### 4.4 本地缓存

```bash
# 缓存位置
$GOPATH/pkg/mod/

# 查看缓存
ls $GOPATH/pkg/mod/cache/download/

# 清理缓存
go clean -modcache
```

## 第五章 实战配置

### 5.1 国内镜像加速

```bash
# 七牛云（推荐）
go env -w GOPROXY=https://goproxy.cn,direct

# 阿里云
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct

# 多个镜像组合
go env -w GOPROXY=https://goproxy.cn,https://mirrors.aliyun.com/goproxy/,direct
```

### 5.2 私有仓库配置

```bash
# 设置私有仓库不走代理
go env -w GOPRIVATE=gitlab.mycompany.com

# 关闭私有仓库校验
go env -w GONOSUMCHECK=gitlab.mycompany.com

# 或使用通配符
go env -w GOPRIVATE=*.mycompany.com
```

### 5.3 典型项目结构

```
myproject/
├── go.mod
├── go.sum
├── main.go
├── internal/       # 内部包（外部不可导入）
├── pkg/            # 可复用包
├── cmd/
│   ├── server/     # 服务入口
│   └── cli/        # CLI 入口
└── vendor/         # 可选
```

### 5.4 replace 用法

```go
// 本地开发
replace github.com/some/pkg => ../local-pkg

// 替换为 fork 版本
replace github.com/original/pkg => github.com/myfork/pkg v1.2.3

// 替换为本地版本
replace github.com/original/pkg => /Users/me/local/pkg
```

## 第六章 常见问题

### 6.1 依赖下载失败

```bash
# 设置代理
go env -w GOPROXY=https://goproxy.cn,direct

# 清除缓存重新下载
go clean -modcache
go mod download
```

### 6.2 go.sum 校验失败

```bash
# 验证依赖
go mod verify

# 重新整理
go clean -modcache
go mod tidy
go mod download
```

### 6.3 升级所有依赖

```bash
go get -u ./...
go mod tidy
```

### 6.4 版本冲突

```bash
# 查看依赖树
go mod graph

# 查看特定依赖的版本
go list -m -versions github.com/gin-gonic/gin
```

## 第七章 三者对比

| 特性 | GOPATH | Vendor | Go Modules |
|------|--------|--------|------------|
| 版本锁定 | ❌ | ✅ | ✅ |
| 官方支持 | ✅ | 半官方 | ✅ 官方标准 |
| 项目位置限制 | 必须在 GOPATH | 无限制 | 无限制 |
| 离线构建 | ❌ | ✅ | ✅（需缓存） |
| Git 仓库体积 | 小 | 大 | 小 |
| 构建可复现 | ❌ | ✅ | ✅ |
| 当前状态 | 已淘汰 | 可选辅助 | 推荐使用 |

## 附录：环境变量速查

```bash
go env GOPATH              # 工作区路径
go env GOMODCACHE          # 模块缓存路径
go env GOPROXY             # 代理地址
go env GOPRIVATE           # 私有仓库域名
go env GONOSUMCHECK        # 不校验的仓库
go env GOSUMDB             # 校验数据库
```
