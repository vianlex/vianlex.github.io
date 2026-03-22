#  Sed 基础用法

> Stream Editor - 流式文本编辑器基础用法

---

## 基础语法

```
sed [选项] '命令' 文件
sed [选项] -e '命令1' -e '命令2' 文件
sed [选项] -f 脚本文件 文件
```

### 命令预览

| 命令 | 说明 | 命令 | 说明 |
|------|------|------|------|
| `p` | 打印 | `d` | 删除 |
| `s` | 替换 | `y` | 字符转换 |
| `i` | 行前插入 | `a` | 行后追加 |
| `c` | 替换整行 | `q` | 退出 |
| `=` | 打印行号 | `r` | 读取文件 |
| `w` | 写入文件 | `l` | 显示不可见字符 |
| `n` | 读取下一行 | `N` | 追加下一行 |
| `P` | 打印至换行 | `D` | 删除至换行 |
| `h` | 模式→保持(覆盖) | `H` | 模式→保持(追加) |
| `g` | 保持→模式(覆盖) | `G` | 保持→模式(追加) |
| `x` | 交换空间 | `:` | 定义标签 |
| `b` | 跳转 | `t` | 替换成功跳转 |
| `T` | 替换失败跳转 | `e` | 执行命令 |
| `F` | 打印文件名 | `z` | 清空模式空间 |
| `Q` | 退出(不打印) | `v` | 检查版本 |

### 选项 (Options)

| 选项 | 说明 |
|------|------|
| `-n` | 静默模式，不自动打印 |
| `-i` | 直接修改原文件 |
| `-i.bak` | 修改前创建备份 |
| `-e` | 执行多条命令 |
| `-f` | 从脚本文件读取命令 |
| `-r` / `-E` | 使用扩展正则 |

```bash
sed -n '1p' file              # 只打印第1行
sed -i 's/old/new/g' file     # 直接修改原文件
sed -i.bak 's/old/new/g' file # 修改并备份
sed -e '1d' -e 's/a/b/' file  # 执行多条命令
sed -E 's/([0-9]+)/[\1]/' file # 使用扩展正则
```

---

## 寻址 (Addressing)

### 行号寻址

| 语法 | 说明 |
|------|------|
| `n` | 第 n 行 |
| `n,m` | 第 n 到 m 行 |
| `n,$` | 第 n 行到最后一行 |
| `$` | 最后一行 |
| `n~m` | 从第 n 行开始，每隔 m 行 |

```bash
sed -n '3p' file              # 第3行
sed -n '1,5p' file            # 1-5行
sed -n '3,$p' file            # 第3行到末尾
sed -n '$p' file              # 最后一行
sed -n '1~2p' file            # 奇数行
sed -n '2~2p' file            # 偶数行
```

### 模式寻址

| 语法 | 说明 |
|------|------|
| `/pattern/` | 匹配 pattern 的行 |
| `/start/,/end/` | 从 start 到 end 的范围 |
| `/pattern/!` | 不匹配 pattern 的行 |

```bash
sed -n '/error/p' file        # 包含 error 的行
sed '/^#/d' file              # 删除 # 开头的行
sed -n '/start/,/end/p' file  # start 到 end 之间
sed '/pattern/!d' file        # 只保留匹配行
```

---

## 打印命令 (p)

| 语法 | 说明 |
|------|------|
| `p` | 打印当前行 (需配合 `-n`) |

```bash
sed -n '1p' file              # 第1行
sed -n '1,5p' file            # 1-5行
sed -n '/error/p' file        # 匹配行
sed -n '/^$/!p' file          # 非空行
```

---

## 删除命令 (d)

| 语法 | 说明 |
|------|------|
| `d` | 删除当前行 |

```bash
sed '3d' file                 # 删除第3行
sed '1,5d' file               # 删除1-5行
sed '/error/d' file           # 删除匹配行
sed '/^$/d' file              # 删除空行
sed '/^#/d' file              # 删除注释
sed '/start/,/end/d' file     # 删除范围
```

---

## 替换命令 (s)

| 语法 | 说明 |
|------|------|
| `s/old/new/` | 替换第一个匹配 |
| `s/old/new/g` | 全局替换 |
| `s/old/new/N` | 替换第 N 个 |
| `s/old/new/i` | 忽略大小写 |
| `s/old/new/p` | 替换后打印 |
| `s/old/new/w file` | 替换行写入文件 |

```bash
sed 's/old/new/' file         # 替换第一个
sed 's/old/new/g' file        # 替换所有
sed 's/old/new/2' file        # 替换第2个
sed 's/old/new/gi' file       # 全局忽略大小写
sed -n 's/old/new/p' file     # 只打印替换行
```

### 分隔符

| 分隔符 | 适用场景 |
|--------|----------|
| `/` | 默认，通用 |
| `#` | 包含 `/` 的内容 |
| `\|` | 路径替换，推荐 |

```bash
sed 's/a/b/' file             # 默认
sed 's#/home#/root#' file     # 路径
sed 's|/usr/local|/opt|g' file # 路径推荐
```

### 捕获组

| 语法 | 说明 |
|------|------|
| `\(pattern\)` | 基本正则捕获 |
| `(pattern)` | 扩展正则捕获 |
| `\1, \2, ...` | 反向引用 |
| `&` | 整个匹配内容 |

```bash
sed 's/\(word\)/[\1]/' file          # 捕获
sed -E 's/(word)/[\1]/' file         # 扩展正则
sed -E 's/(\w+) (\w+)/\2 \1/' file   # 交换位置
sed 's/word/[&]/g' file              # 包裹匹配
sed 's/.*/& &/' file                 # 复制行
```

### 大小写转换 (GNU sed)

| 转义 | 说明 |
|------|------|
| `\U` | 转大写 |
| `\L` | 转小写 |
| `\u` | 下一个字符大写 |
| `\l` | 下一个字符小写 |
| `\E` | 结束转换 |

```bash
sed 's/.*/\U&/' file          # 全部大写
sed 's/.*/\L&/' file          # 全部小写
sed 's/\b\w/\u&/g' file       # 首字母大写
```

---

## 插入命令 (i)

| 语法 | 说明 |
|------|------|
| `n i\text` | 在第 n 行前插入 |
| `/pattern/ i\text` | 在匹配行前插入 |

```bash
sed '3i\新行' file            # 第3行前插入
sed '$i\末尾前插入' file      # 最后一行前插入
sed '/pattern/i\插入' file    # 匹配行前插入
```

---

## 追加命令 (a)

| 语法 | 说明 |
|------|------|
| `n a\text` | 在第 n 行后追加 |
| `/pattern/ a\text` | 在匹配行后追加 |

```bash
sed '3a\新行' file            # 第3行后追加
sed '$a\末尾追加' file        # 末尾追加
sed '/pattern/a\追加' file    # 匹配行后追加
```

---

## 替换整行命令 (c)

| 语法 | 说明 |
|------|------|
| `n c\text` | 替换第 n 行 |
| `/pattern/ c\text` | 替换匹配行 |

```bash
sed '3c\新内容' file          # 替换第3行
sed '/pattern/c\新内容' file  # 替换匹配行
sed '1,5c\替换内容' file      # 1-5行替换为一行
```

---

## 字符转换命令 (y)

| 语法 | 说明 |
|------|------|
| `y/源字符/目标字符/` | 逐字符转换 |

```bash
sed 'y/abc/ABC/' file         # a→A, b→B, c→C
sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' file  # 转大写
```

---

## 退出命令 (q)

| 语法 | 说明 |
|------|------|
| `n q` | 处理完第 n 行后退出 |
| `/pattern/ q` | 匹配后退出 |

```bash
sed '5q' file                 # 前5行后退出
sed '/pattern/q' file         # 匹配后退出
sed -n '/pattern/{p;q}' file  # 只打印第一个匹配
```

---

## 行号命令 (=)

| 语法 | 说明 |
|------|------|
| `=` | 打印当前行号 |

```bash
sed = file                    # 每行前打印行号
sed -n '/pattern/=' file      # 匹配行号
sed -n '$=' file              # 总行数
```

---

## 读写命令 (r / w)

| 命令 | 说明 |
|------|------|
| `n r file` | 第 n 行后读入文件 |
| `/pattern/ r file` | 匹配行后读入文件 |
| `n w file` | 第 n 行写入文件 |
| `/pattern/ w file` | 匹配行写入文件 |

```bash
sed '3r other.txt' file       # 第3行后插入
sed '$r other.txt' file       # 末尾插入
sed '1,5w out.txt' file       # 1-5行写入
sed '/error/w err.txt' file   # 匹配行写入
```

---

## 多行处理 (N / P / D)

| 命令 | 说明 |
|------|------|
| `N` | 追加下一行到模式空间 |
| `P` | 打印到第一个 `\n` 的内容 |
| `D` | 删除到第一个 `\n` 的内容 |

```bash
sed 'N;s/\n/ /' file          # 合并两行
sed -n 'N;P' file             # 奇数行
sed '/^$/N;/^\n$/d' file      # 删除连续空行
```

---

## 保持空间 (Hold Space)

| 命令 | 说明 |
|------|------|
| `h` | 模式空间 → 保持空间 (覆盖) |
| `H` | 模式空间 → 保持空间 (追加) |
| `g` | 保持空间 → 模式空间 (覆盖) |
| `G` | 保持空间 → 模式空间 (追加) |
| `x` | 交换两个空间 |

```bash
sed 'G' file                  # 每行后加空行
sed '1!G;h;$!d' file          # 逆序输出
sed 'x;$G' file               # 末行前置
```

---

## 标签与跳转

| 命令 | 说明 |
|------|------|
| `:label` | 定义标签 |
| `b label` | 跳转到标签 |
| `t label` | 替换成功则跳转 |
| `T label` | 替换失败则跳转 |

```bash
sed ':a;s/old/new/g;ta' file  # 重复替换
sed ':a;N;$!ba;s/\n/ /g' file # 合并所有行
```

---

## 正则表达式

### 基础正则 (BRE)

| 元字符 | 说明 |
|--------|------|
| `.` | 任意单字符 |
| `*` | 0 或多个 |
| `^` | 行首 |
| `$` | 行尾 |
| `[]` | 字符集 |
| `[^]` | 否定字符集 |
| `\{n,m\}` | 重复次数 |

### 扩展正则 (ERE - 使用 -E)

| 元字符 | 说明 |
|--------|------|
| `+` | 1 或多个 |
| `?` | 0 或 1 个 |
| `()` | 分组 |
| `\|` | 或 |
| `{n,m}` | 重复次数 |
| `\b` | 单词边界 |
| `\w` | 单词字符 |
| `\s` | 空白字符 |

```bash
sed 's/a.c/X/' file           # 任意字符
sed 's/a*/X/' file            # 0或多
sed '/^start/p' file          # 行首
sed '/end$/p' file            # 行尾
sed 's/[abc]/X/g' file        # 字符集
sed -E 's/a+/X/' file         # 1或多
sed -E 's/(cat|dog)/X/' file  # 或
sed -E 's/\bword\b/X/' file   # 单词边界
```

---

## 常用代码片段

### 清理

```bash
sed '/^$/d' file                          # 删除空行
sed '/^#/d' file                          # 删除注释
sed 's/^[[:space:]]*//' file              # 删除行首空格
sed 's/[[:space:]]*$//' file              # 删除行尾空格
sed 's/^[[:space:]]*//;s/[[:space:]]*$//' file  # 删除首尾空格
```

### 替换

```bash
sed 's/old/new/g' file                    # 全局替换
sed "s/pattern/$var/" file                # 变量替换
sed 's|/old/path|/new/path|g' file        # 路径替换
sed -E 's/([0-9]+)/[\1]/g' file           # 捕获数字
sed 's/^/# /' file                        # 添加注释
```

### 转换

```bash
sed 's/$/\r/' file                        # Unix → Windows
sed 's/\r$//' file                        # Windows → Unix
sed 's/<[^>]*>//g' file                   # 去除HTML标签
sed 's/\t/    /g' file                    # Tab → 空格
```

### 提取

```bash
sed -n '1,5p' file                        # 提取1-5行
sed -n '/start/,/end/p' file              # 提取范围
sed -n '1p;$p' file                       # 首尾行
sed = file | sed 'N;s/\n/\t/'             # 添加行号
```

### 合并

```bash
sed 'N;s/\n/ /' file                      # 合并两行
sed ':a;N;$!ba;s/\n/ /g' file             # 合并所有行
sed 'N;N;s/\n/ /g' file                   # 合并三行
```

---

## 常见陷阱

| 问题 | 原因 | 解决 |
|------|------|------|
| macOS `-i` 报错 | BSD sed 需要参数 | `sed -i ''` |
| 变量不展开 | 单引号 | 双引号或混合 |
| 重复打印 | 缺 `-n` | `-n` + `p` |
| 路径转义 | `/` 分隔符 | 换 `#` 或 `\|` |

```bash
# macOS 正确写法
sed -i '' 's/a/b/' file

# 变量替换正确写法
sed "s/pattern/$var/" file
sed 's/pattern/'"$var"'/' file

# 路径推荐写法
sed 's|/old|/new|g' file
```

---

## Sed vs Awk

| 特性 | Sed | Awk |
|------|-----|-----|
| 擅长 | 替换、删除 | 统计、计算 |
| 变量 | 不支持 | 支持 |
| 数学 | 不支持 | 支持 |
| 条件 | 有限 | 完整 |

```bash
# 组合使用
cat log.txt | sed 's/\[.*\]//' | awk '{print $1, $NF}'
grep pattern file | sed 's/a/b/' | awk '{sum+=$1} END{print sum}'
```

---

*v1.0 | 2026-03-22*
