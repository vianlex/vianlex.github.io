# Go 包导入说明

## 一、import 导入方式

### 1.1 匿名导入

```go
import "fmt"              // 标准导入
import f "fmt"            // 别名导入
import . "fmt"            // 点号导入
import _ "fmt"            // 下划线导入
```

| 方式 | 语法 | 作用 |
|------|------|------|
| 标准导入 | `import "pkg"` | 正常使用包成员 |
| 别名导入 | `import f "pkg"` | 解决命名冲突 |
| 点号导入 | `import . "pkg"` | 直接使用成员，无需包前缀 |
| 下划线导入 | `import _ "pkg"` | 仅执行 init，不引用任何成员 |

### 1.2 点号导入

```go
import . "fmt"

func main() {
    Println("Hello")  // 直接使用，无需 fmt.
}
```

### 1.3 下划线导入

```go
import _ "github.com/go-sql-driver/mysql"
```

常用于：
- 注册数据库驱动
- 插件初始化

### 1.4 别名导入

```go
import f "fmt"

func main() {
    f.Println("Hello")
}
```

### 1.5 匿名导入接口检查

```go
import "io"

var _ io.Reader = (*MyReader)(nil)
```

编译时检查 `*MyReader` 是否实现了 `io.Reader` 接口。

---

## 二、导入路径

### 2.1 三种路径类型

```
┌─────────────────────────────────────────────┐
│                  import 路径                 │
├──────────────┬──────────────┬───────────────┤
│   标准库     │   本地包     │   远程包      │
│              │              │               │
│  go 标准库   │ go.mod 模块名 │ github.com/  │
└──────────────┴──────────────┴───────────────┘
```

### 2.2 标准库

```go
import "fmt"
import "net/http"
```

**查找路径：**

```bash
GOROOT/src/fmt
GOROOT/src/net/http
```

### 2.3 本地包

```go
// go.mod
module github.com/user/project

// main.go
import "github.com/user/project/internal/service"
```

**查找规则：**
1. 读取 go.mod 第一行作为模块名
2. 模块名作为项目根路径
3. 从根路径查找包目录

### 2.4 远程包

```go
import "github.com/gin-gonic/gin"
```

**查找路径：**

```bash
$GOMODCACHE/github.com/gin-gonic/gin@v1.9.1/

# 查看缓存路径
go env GOMODCACHE
```

---

## 三、go.mod 文件结构

```go
module github.com/user/project

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
)

require (
    golang.org/x/net v0.17.0 // indirect
)

replace github.com/old => ../local-pkg

exclude github.com/bad/pkg v1.2.3
```

| 指令 | 说明 |
|------|------|
| `module` | 模块路径（唯一标识） |
| `require` | 依赖声明 |
| `indirect` | 间接依赖 |
| `replace` | 替换依赖（本地开发用） |
| `exclude` | 排除某版本 |

---

## 四、go.sum 文件

```
github.com/gin-gonic/gin v1.9.1 h1:Hash值
github.com/gin-gonic/gin v1.9.1/go.mod h1:Hash值
```

**作用：** 校验依赖完整性，防止篡改

---

## 五、版本选择（MVS）

**Minimum Version Selection**：选择满足所有需求的最小版本

```
项目 A → 依赖 gin v1.8.0
项目 A → 依赖 B → 依赖 gin v1.9.1

→ 最终选择 v1.9.1
```

**语义化版本：**

```
v1.9.1
 │ │ └── Patch：bug 修复
 │ └──── Minor：新功能
 └────── Major：破坏性变更
```

**v2+ 必须改路径：**

```go
import "github.com/gin-gonic/gin/v2"
```

---

## 六、init 初始化

### 6.1 执行顺序

```
import → const → var → init() → main()
```

### 6.2 init 函数

```go
func init() {
    // 初始化逻辑
}
```

**特点：**
- 无参数无返回值
- 自动执行
- 可定义多个，按出现顺序执行

---

## 七、循环导入

### 7.1 问题

```go
// a.go
import "b"
func A() {}

// b.go
import "a"  // 循环依赖！
func B() {}
```

### 7.2 解决

- 提取公共接口到第三个包
- 依赖注入
- 重构代码结构

---

## 八、internal 包

**可见性规则：** 只能被父包及其子包访问

```
project/
├── cmd/main.go      ✓ 可访问 internal/pkg
└── internal/pkg/   ✓ 可访问 internal/utils
    └── utils/

external/            ✗ 不可访问 internal/pkg
```

---

## 九、常用命令

```bash
# 初始化
go mod init github.com/user/project

# 添加依赖
go get github.com/gin-gonic/gin
go get github.com/gin-gonic/gin@v1.9.1

# 下载依赖
go mod download

# 整理依赖
go mod tidy

# 查看依赖
go list -m all

# 替换依赖
go mod edit -replace github.com/old=github.com/new

# 清理缓存
go clean -modcache
```

---

## 十、代理配置

```bash
# 国内代理
go env -w GOPROXY=https://goproxy.cn,direct

# 私有仓库
go env -w GOPRIVATE=gitlab.mycompany.com

# 关闭校验
go env -w GONOSUMCHECK=gitlab.mycompany.com
```

---

## 十一、环境变量

| 变量 | 说明 |
|------|------|
| `GOPATH` | 工作区路径 |
| `GOMODCACHE` | 模块缓存路径 |
| `GOPROXY` | 代理地址 |
| `GOPRIVATE` | 私有仓库 |
