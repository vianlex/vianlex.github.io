# Cmd 常用命令


## type 查看文件内容

1、基本用法

```bash
# 查看单个文件内容（等效 cat file.txt）
type 文件名.txt

# 拼接多个文件内容并显示（等效 cat file1.txt file2.txt）
type 文件1.txt 文件2.txt

# 拼接多个文件并写入新文件（等效 cat file1.txt file2.txt > new.txt）
type 文件1.txt 文件2.txt > 合并后的文件.txt

# 追加多个文件内容到已有文件（等效 cat file1.txt >> existing.txt）
type 文件1.txt >> 已有文件.txt

# 禁止换行/空行压缩（默认 type 会压缩连续空行，加 /P 保留原格式）
type /P 带空行的文件.txt

# 文件路径中有空格时，需用双引号包裹
type "C:\我的文件\test.txt"
```

2、乱码问题处理

```bash
# 切换 CMD 编码为 UTF-8
chcp 65001

# 再查看 UTF-8 编码的文件
type utf8编码的文件.txt
```


## more 查看文件内容

1、基础命令

```bash
# 查看命令用户 
more /?

# 分页查看文件（按 Enter 翻行，按 Space 翻页，按 Q 退出）
more 大文件.txt

# 从 100 行开始查看
more +100 文件.txt

# 结合 type 实现分页（等效 cat 大文件.txt | more）
type 大文件.txt | more
```

2、文件显示乱码问题

```bash
# 切换 CMD 编码为 UTF-8
chcp 65001
# 再查看 UTF-8 编码的文件
more utf8编码的文件.txt
```

## findstr 查找文件内容（类型 grep 命令）

1、findstr 查看文件内容

```bash
# 查看文件内容，并显示行号（等价于 linux 下的 cat -n file.text）
findstr /N "^" 文件名.txt
```

2、查找文件内容

```bash
# 1. 在单个文件中搜索指定文本
findstr "error" app.log

# 2. 在多个文件中搜索（当前目录所有 .txt 文件）
findstr "warning" *.txt

# 3. 搜索时忽略大小写
findstr /i "ERROR" app.log

# 4. 显示匹配行的行号
findstr /n "success" app.log

# 5. 从管道接收内容并搜索（比如搜索目录中的特定文件）
dir | findstr /i "test"

```

3、查找文件内容，输出乱码问题

```bash
# 将 Cmd 切换为 UTF-8 编码
chcp 65001  
# 查找内容，并忽略大小写
findstr /i "中文" utf8文件.txt
```

## netstat 查看端口

1、基础用法

```bash

# 查看 8080 端口占用情况
netstat -nao | findstr ":8080"
```