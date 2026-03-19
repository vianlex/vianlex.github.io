# PowerShell 常用命令

## PowerShell 乱码问题

1、解决方案（临时生效，重启失效）

```bash
# 1. 查看当前控制台编码
[Console]::OutputEncoding

# 2. 设置控制台编码为 UTF-8（推荐）
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# 或设置为 GBK（兼容老文件）
[Console]::OutputEncoding = [System.Text.Encoding]::GetEncoding("GBK")
```

2、创建/编辑 PowerShell 配置文件（永久生效）

```bash
# 1. 查看用户配置文件是否存在，输出目录则说明存在
$PROFILE

# 2. 如果配置文件不存在，则创建配置文件
New-Item -Type File -Path $PROFILE -Force 

# 3. 通过 notepad 记事本打开配置文件
notepad $PROFILE

# 4. 在配置文件中添加以下内容即可

[Console]::OutputEncoding = [System.Text.Encoding]::UTF8 # 设置控制台输出编码为 UTF-8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8 # 设置控制台输入编码为 UTF-8

$OutputEncoding = [System.Text.Encoding]::UTF8  # 设置外部命令输出编码为 UTF-8

$PSDefaultParameterValues['Get-Content:Encoding'] = 'UTF8' # 设置 Get-Content 默认编码为 UTF-8（PowerShell 7+）
$PSDefaultParameterValues['Out-File:Encoding'] = 'UTF8' # 设置 Out-File 默认编码为 UTF-8（PowerShell 7+）

```