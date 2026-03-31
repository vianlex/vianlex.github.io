# Go 导入包说明

## 第一章 import 使用场景

### 1.1 两种导入类型

| 类型 | 示例 | 查找路径 |
|------|------|----------|
| 标准库 | `import "fmt"` | `GOROOT/src` |
| 模块包 | `import "github.com/gin-gonic/gin"` | `GOMODCACHE` |

## 第二章 标准库导入

```go
import "fmt"
import "net/http"
```

标准库实际查找路径：

```bash
GOROOT/src/fmt
GOROOT/src/net/http
```

## 第三章 模块包导入

### 3.1 模块包类型

```
模块包
├── 本地项目包    # 自定义包
└── 第三方包      # 外部依赖
```

### 3.2 导入规则

```
import 路径 = go.mod 模块名 + 包目录
```

**示例：**

```
模块名：github.com/abc/demo
包目录：internal/service
import 路径：github.com/abc/demo/internal/service
```

### 3.3 本地项目包导入

**目录结构：**

```
mydemo/
├── go.mod
├── main.go
└── user/
    └── user.go
```

**go.mod：**

```go
module github.com/xxx/demo
```

**main.go 中引入 user 包：**

```go
import "github.com/xxx/demo/user"
```

**查找流程：**

1. 读取 go.mod 第一行作为模块名
2. 模块名作为项目根路径别名
3. 从项目根目录开始查找包目录

### 3.4 第三方包导入

```go
import "github.com/gin-gonic/gin"
```

**实际路径：**

```bash
GOMODCACHE/github.com/gin-gonic/gin@v1.9.1/

# 查看 GOMODCACHE 路径
go env GOMODCACHE
```

## 第四章 跨项目引用

### 4.1 方式一：GitHub 引用

```go
import "github.com/xxx/demo/user"
```

### 4.2 方式二：replace 本地路径

```bash
go mod edit -replace=github.com/xxx/demo/user=/path/to/local/demo/user
```

## 第五章 Go Mod 常用命令

### 5.1 初始化

```bash
mkdir go-demo && cd go-demo
go mod init github.com/xxx/go-demo
```

### 5.2 依赖管理

```bash
# 添加依赖（自动写入 go.mod）
go get github.com/gin-gonic/gin

# 下载所有依赖
go mod download

# 整理依赖（删除未用、补充缺失）
go mod tidy

# 查看依赖列表
go list -m all

# 替换依赖
go mod edit -replace github.com/old=github.com/new
```

### 5.3 代理配置

```bash
# 国内代理
go env -w GOPROXY=https://goproxy.cn,direct

# 私有仓库不走代理
go env -w GOPRIVATE=gitlab.mycompany.com
```

## 第六章 导入总结

| 导入类型 | 查找路径 |
|----------|----------|
| 标准库 | `GOROOT/src` |
| 本地项目 | 以 go.mod 模块名为根目录 |
| 第三方包 | `GOMODCACHE` |
