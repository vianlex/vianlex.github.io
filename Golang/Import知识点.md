# Import 知识点

## Go import 两种使用场景

1. import 标准库包（fmt, net/http）
2. import 模块包（本地项目或者第三方包） 

## import 标准库包

```bash
import "fmt"
import "net/http"
```

import 标准库包，实际查找路径为：`GOROOT/src`

```bash
# GOROOT 为 Go 的安装根目录
GOROOT/src/fmt
GOROOT/src/net/http
```

## import 模块包（本地项目包 + 第三方包）

### import 本地项目包

import 规则：`import 路径 = go.mod 里的模块名 + 从项目根出发的包目录`, 如下例子：

```txt
项目go.mod声明的模块名：github.com/abc/demo
包目录：internal/service
import 必须写为： import github.com/abc/demo/internal/service
```

本地项目不同包之前 import 包查找说明：

1. 看当前项目根目录有没有 go.mod
2. 读取第一行模块名
3. import 时，把模块名当作 “项目根路径别名”
4. 直接从项目根开始找目录

#### import 本地项目包示例

1. go.mod 第一行（模块名）

```bash
#go.mod 文件第一行
module github.com/xxx/demo
```

2. 本地项目目录结构

```bash
mydemo/
├── go.mod
├── main.go
└── user/        # 这是一个包
    └── user.go
```

3. import 写法

```bash
# 如在 main.go 中需要引入 user 包时，写法如下
import "github.com/xxx/demo/user"
```

4. 实际引入路径

```bash
# 
本地项目目录/user/
```

### import 第三方包

1. import 写法

```bash
import "github.com/gin-gonic/gin"
```

2. 真实路径

```bash
GOMODCACHE/github.com/gin-gonic/gin@v1.9.1/
# 通过以下命令可以查看 GOMODCACHE 的路径
go env GOMODCACHE
```

## import 总结

- 标准库 → 去 GOROOT/src 找
- 本地项目 → 以 go.mod 模块名为根目录找
- 第三方包 → 去模块缓存 GOMODCACHE 里找

## 本地不同项目引用

### 第一种方式

将项目上传到 github 然后直接引用即可。

```bash
import "github.com/xxx/demo/user"
```

### 第二种方式

通过 go mod edit + 本地路径方式实现

```bash
# 通过 replace 替换成实际的应用路径
go mod edit -replace=github.com/xxx/demo/user=/xx/xx/demo/user
```

## Go Mod 常用命令

```bash
# 初始化模块
mkdir go-demo && cd go-demo
go mod init github.com/xxx/go-demo

# 下载指定依赖，并将依赖添加到 go.mod 文件中
go get github.com/gin-gonic/gin

# 根据 go.mod 下载所有依赖
go mod download

# 整理依赖
go mod tidy

# 查看依赖
go list -m all

# 替换依赖，将 github.com/old 依赖替换成新的 github.com/new 依赖
go mod edit -replace github.com/old=github.com/new

# 代理配置（国内代理）
go env GOPROXY=https://goproxy.cn,direct
```
