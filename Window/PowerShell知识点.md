# PowerShell 基础知识点

## PowerShell 的核心概念

### Cmdlet：最小命令单元

Cmdlet（发音为 command-let）是 PowerShell 中最基本的命令单元，它们都是基于 .NET 的类，运行后会返回一个或多个 .NET 对象。

每个 Cmdlet 都遵循【动词-名词】命名规范，如下：

| Cmdlet 命令 | 含义描述 |
| :--- | :--- |
| `Get-Command` | 查看所有可用命令 |
| `Get-Help` | 查看命令的帮助信息 |
| `Get-Process` | 获取进程列表 |
| `Get-Service` | 获取服务列表 |
| `Start-Service` | 启动服务 |
| `Stop-Service` | 停止服务 |
| `Set-ExecutionPolicy` | 设置执行策略 |
| `New-Item` | 创建新文件或文件夹 |
| `Remove-Item` | 删除文件或文件夹 |
| `Copy-Item` | 复制文件或文件夹 |
| `Move-Item` | 移动文件或文件夹 |
| `Clear-Host` | 清屏，类似于 cls |

### 对象管道（Object Pipeline）

PowerShell 中的管道（Pipeline）与 Linux/Unix Shell（如 Bash）的管道不同，PowerShell 的管道传递的不是文本字符串，而是 .NET 对象（Objects）。
意味着数据在命令之间流动时，保留了其属性（Properties）和方法（Methods），无需像传统 Shell 那样使用 grep、awk 或 cut 来解析文本，简单使用例子如下：

```bash
# 1. Get-Process 返回 System.Diagnostics.Process 对象，表示的是系统进程对象列表
# 2. 管道 | 将完整的对象传递给 Where-Object
# 3. Where-Object 直接访问对象的 .WorkingSet64 属性进行数字比较
# 4. 管道 | 将过滤后的对象传递给 Select-Object
# 5. Select-Object 提取特定的属性生成新对象
# $_ 代表管道中当前正在处理的对象
Get-Process | 
    Where-Object { $_.WorkingSet64 -gt 100MB } | 
    Select-Object Name, WorkingSet64
    
# 查看对象有哪些属性
Get-Process | Get-Member
```

### Provider 与 PSDrive 抽象资源对象

PowerShell 的 Provider（提供程序） 和 PSDrive（PowerShell 驱动器），将「非文件系统资源」抽象成「驱动器」的形式。让我们可以通过 cd、dir、Get-ChildItem 等命令操作注册表、证书、环境变量等资源。

核心特性：
- 统一操作语法：无论操作文件、注册表还是证书，都用 Get-ChildItem/Set-Item/New-Item 等通用 cmdlet；
- 资源抽象：将非文件资源映射为「目录/项」结构（如注册表项 = 目录，注册表值 = 文件）；
- 可扩展：支持自定义 Provider（如第三方开发的云存储 Provider）。

常见的 PSProvider 类型：

| Provider 提供程序 | 作用 | 抽象资源驱动 PSDrive | 描述说明 |
| :--- | :--- | :--- | :--- | 
| FileSystem	| 操作文件 / 目录 | 	C:、D:、E: 等 | 将 FileSystem 程序，抽象成驱动器，让我们可以通过操作驱动器的方式，操作 FileSystem 程序类 |
| Environment	| 操作环境变量	  |   Env:         | 将 Environment 程序，抽象成驱动器，让我们可以通过操作驱动器的方式，操作 Environment 程序类|
| Registry    | 操作 Windows 注册表| HKLM:（本地机器）、HKCU:（当前用户）| 查看注册表中的 \SOFTWARE\Microsoft\Windows 信息 Get-ChildItem HKLM:\SOFTWARE\Microsoft\Windows | 
| Variable	  | 操作 PowerShell 变量	| Variable: |  查看环境变量中的 Path 变量，Get-Item Env:\PATH |
| Alias	      | 操作 PowerShell 别名	|  Alias:  |  通过操作目录的方式：Get-Item Alias:\dir 查看 dir 是哪个命令的别名|
| Certificate	| 操作证书存储	|    Cert: | 通过操作目录的方式，访问 Certificate 程序属性：Get-ChildItem Cert:\LocalMachine\My 查看本地机器证书 |

#### PSDrive 抽象驱动器基础命令

1. 创建自定义 PsDrive 抽象资源驱动器

```bash
# 基础语法：
New-PSDrive -Name <名字> -PSProvider <提供程序> -Root <实际路径>

# 映射一个长目录为抽象驱动器
New-PSDrive -Name dev -PSProvider FileSystem -Root "D:\workspace\project\server"

# 映射网络共享文件夹为抽象驱动器
New-PSDrive -Name nas -PSProvider FileSystem -Root "\\192.168.1.100\share"

# 映射当前用户桌面为抽象驱动器
New-PSDrive -Name desk -PSProvider FileSystem -Root "$([Environment]::GetFolderPath('Desktop'))"
```

2. 查看抽象驱动器

```bash
# 查看所有驱动器
Get-PSDrive 

# 查看文件系统类型的抽象驱动器
Get-PSDrive -PSProvider FileSystem
```

3. 删除自定义抽象驱动器

```bash 
Remove-PSDrive -Name dev
```

5. 让自定义 PSDrive 永久生效（重启不消失）

```bash
# 打开用户配置文件
notepad $PROFILE
# 如果用户配置文件不存在，则通过命令创建
New-Item -Path $profile -ItemType File -Force
# 将自动抽象驱动器，写入用户配置文件，即可永久生效
New-PSDrive -Name dev -PSProvider FileSystem -Root "D:\workspace\project" -Scope Global
New-PSDrive -Name desk -PSProvider FileSystem -Root "$([Environment]::GetFolderPath('Desktop'))" -Scope Global
```

#### 操作 PsDrive 抽象驱动器例子

1、将系统环境变量抽象成 `Env:` 按驱动器和文件系统的方式访问

```bash
# 切换到环境变量驱动器 Env:
cd Env:

# 通过 dir 列出当前 Env: 目录下的所有环境变量
dir 
# 或者 Get-ChildItem 列出当前 Env: 目录下的所有环境变量

# 以文件驱动器的方式查看，环境变量 Path 的值
dir Env:\Path
# 或
Get-ChildItem Env:\Path

# 简写方式读取环境便利
$env:PATH

```
