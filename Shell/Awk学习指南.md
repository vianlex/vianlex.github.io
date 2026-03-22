# AWK 学习指南

> AWK 学习指南 | 涵盖基础到进阶

---

## 目录

1. [AWK 简介](#1-awk-简介)
2. [工作原理](#2-工作原理)
3. [内置变量](#3-内置变量)
4. [基本用法](#4-基本用法)
5. [条件过滤](#5-条件过滤)
6. [格式化输出](#6-格式化输出)
7. [运算与统计](#7-运算与统计)
8. [数组](#8-数组)
9. [字符串函数](#9-字符串函数)
10. [控制流程](#10-控制流程)
11. [分隔符](#11-分隔符)
12. [多文件处理](#12-多文件处理)
13. [函数](#13-函数)
14. [实战案例](#14-实战案例)
15. [性能优化](#15-性能优化)
16. [常见问题](#16-常见问题)
17. [速查表](#17-速查表)

---

## 1. AWK 简介

### 1.1 什么是 AWK？

AWK 是一种**强大的文本处理工具**，诞生于 1977 年，由贝尔实验室的三位计算机科学家发明：

- **A**ho
- **W**einberger  
- **K**ernighan

它的名字就是这三位姓氏首字母的组合。

### 1.2 AWK 能做什么？

| 场景 | 示例 |
|------|------|
| 提取列 | 从日志中提取 IP 地址 |
| 数据统计 | 计算总和、平均值、最大值 |
| 格式化 | 生成表格、报表 |
| 过滤 | 筛选符合条件的行 |
| 转换 | CSV 转 JSON、文本格式化 |
| 分析 | 访问日志、服务器日志分析 |

### 1.3 AWK 变体

| 变体 | 说明 |
|------|------|
| AWK | 原始版本 |
| NAWK | New AWK，增强版 |
| GAWK | GNU AWK，Linux 默认版本，最完善 |
| MAWK | Mike AWK，速度最快 |

```bash
# 查看版本
awk --version
gawk --version
```

---

## 2. 工作原理

### 2.1 核心概念

AWK 处理数据的方式是：**逐行读取 → 按列拆分 → 处理字段**

```
┌─────────────────────────────────────────────────────────────┐
│                        数据流                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  输入文件                                                   │
│  ┌─────────────────────┐                                  │
│  │ 第1行: Tom,95,Math  │ ← 记录 1 (Record 1)             │
│  │ 第2行: Alice,88,Eng │ ← 记录 2 (Record 2)             │
│  │ 第3行: Bob,92,His   │ ← 记录 3 (Record 3)             │
│  └─────────────────────┘                                  │
│              ↓                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  按行分隔符 (RS) 拆分                                │   │
│  │  默认：换行符 \n                                    │   │
│  └─────────────────────────────────────────────────────┘   │
│              ↓                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  按字段分隔符 (FS) 拆分                              │   │
│  │  默认：空格、Tab                                    │   │
│  │                                                     │   │
│  │  Tom,95,Math  →  $1="Tom" $2="95" $3="Math"     │   │
│  └─────────────────────────────────────────────────────┘   │
│              ↓                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  执行 pattern { action }                            │   │
│  └─────────────────────────────────────────────────────┘   │
│              ↓                                             │
│  输出                                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 处理流程图

```
                    ┌─────────────────────┐
                    │    程序开始          │
                    │    BEGIN { }        │ ← 只执行1次
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
         ▼                     ▼                     ▼
   ┌──────────┐         ┌──────────┐         ┌──────────┐
   │ 读取第1行 │         │ 读取第2行│         │ 读取第N行│
   └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
         │                     │                     │
         ▼                     ▼                     ▼
   ┌──────────┐         ┌──────────┐         ┌──────────┐
   │ 匹配pattern?│        │ 匹配pattern?│        │ 匹配pattern?│
   └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
         │                     │                     │
    是───▼───────────    是───▼───────────    是───▼───────────
    │ 执行 action    │    │ 执行 action    │    │ 执行 action    │
    └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
         │                     │                     │
         └─────────────────────┼─────────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    程序结束          │
                    │    END { }          │ ← 只执行1次
                    └─────────────────────┘
```

### 2.3 执行顺序

```bash
awk '
BEGIN {
    print "=== 初始化 ==="     # 1. 先执行（1次）
}
{
    print $1                    # 2. 逐行执行（每行1次）
}
END {
    print "=== 完成 ==="        # 3. 最后执行（1次）
}
' file.txt
```

---

## 3. 内置变量

### 3.1 字段变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `$0` | 当前整行 | `Tom,95,Math` |
| `$1` | 第1列 | `Tom` |
| `$2` | 第2列 | `95` |
| `$3` | 第3列 | `Math` |
| `$NF` | 最后1列 | `$3` |
| `$(NF-1)` | 倒数第2列 | `$2` |
| `$N` | 第N列 | 动态 |

**示例：**
```bash
# 文件: Tom,95,Math
echo "Tom,95,Math" | awk -F',' '{ 
    print "$0 =", $0    # Tom,95,Math
    print "$1 =", $1    # Tom
    print "$2 =", $2    # 95
    print "$3 =", $3    # Math
    print "$NF =", $NF  # Math
}'
```

### 3.2 记录变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `NR` | 当前行号（多文件累计） | `1, 2, 3...` |
| `FNR` | 当前文件行号 | `1, 2, 3...` |
| `NF` | 当前行字段数 | `3` |
| `FILENAME` | 当前文件名 | `data.txt` |

**NR vs FNR：**
```bash
# 文件1: 3行, 文件2: 2行

awk '{ print "NR=" NR, "FNR=" FNR }' file1.txt file2.txt
# 输出：
# NR=1 FNR=1  ← file1第1行
# NR=2 FNR=2  ← file1第2行
# NR=3 FNR=3  ← file1第3行
# NR=4 FNR=1  ← file2第1行 (NR累加，FNR重置)
# NR=5 FNR=2  ← file2第2行
```

### 3.3 分隔符变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `FS` | 输入字段分隔符 | 空格、Tab |
| `OFS` | 输出字段分隔符 | 空格 |
| `RS` | 输入记录分隔符 | 换行符 |
| `ORS` | 输出记录分隔符 | 换行符 |
| `FPAT` | 字段正则匹配 | — |

**示例：**
```bash
# 改变输入输出分隔符
awk 'BEGIN { FS=","; OFS=" | " } { print $1, $2 }' data.csv

# 输出：Tom | 95
```

### 3.4 环境变量

| 变量 | 说明 |
|------|------|
| `ENVIRON["VAR"]` | 读取环境变量 |

```bash
# 读取环境变量
awk 'BEGIN { print ENVIRON["HOME"] }'
awk 'BEGIN { print ENVIRON["PATH"] }'
```

---

## 4. 基本用法

### 4.1 最小语法

```bash
# 基本结构
awk 'pattern { action }' file

# 省略 pattern（匹配所有）
awk '{ print }' file

# 省略 action（默认打印整行）
awk '/pattern/' file

# 省略两者
awk '1' file    # 等同于 print $0
awk '0' file    # 不输出任何内容
```

### 4.2 多种调用方式

```bash
# 1. 处理文件
awk '{ print }' file.txt

# 2. 管道输入
cat file.txt | awk '{ print }'

# 3. 多行脚本
awk '
{
    print $1
}
' file.txt

# 4. 脚本文件
awk -f script.awk file.txt

# 5. 命令行变量
awk -v name=Tom '{ print name, $1 }' file.txt

# 6. 从变量读取
awk '{ print }' <<< "Hello AWK"
```

### 4.3 打印操作

```bash
# 打印整行
awk '{ print }' file
awk '{ print $0 }' file

# 打印多列（逗号分隔）
awk '{ print $1, $2 }' file

# 打印无分隔符
awk '{ print $1 $2 }' file

# 打印字符串
awk '{ print "Hello", "World" }' file

# 打印变量
awk '{ name="Tom"; print name }' file

# 打印转义字符
awk 'BEGIN { print "Line1\nLine2" }'
awk 'BEGIN { print "Tab\tColumn" }'
```

---

## 5. 条件过滤

### 5.1 行级条件

```bash
# 数字比较
awk '$2 > 90' file          # 第2列 > 90
awk '$2 >= 90' file         # 第2列 >= 90
awk '$2 < 90' file          # 第2列 < 90
awk '$2 <= 90' file         # 第2列 <= 90
awk '$2 == 90' file         # 第2列 == 90
awk '$2 != 90' file         # 第2列 != 90

# 字符串比较
awk '$1 == "Tom"' file      # 第1列等于Tom
awk '$1 != "Tom"' file      # 第1列不等于Tom
awk '$1 < "Tom"' file       # 字典序比较
```

### 5.2 复合条件

```bash
# 且 (AND)
awk '$2 > 90 && $3 == "Math"' file

# 或 (OR)
awk '$2 > 90 || $2 < 60' file

# 非 (NOT)
awk '!($2 > 90)' file
awk '$2 !~ /[0-9]/' file     # 第2列不包含数字
```

### 5.3 正则匹配

```bash
# 包含
awk '/Math/' file           # 整行包含Math
awk '$1 ~ /Math/' file      # 第1列包含Math

# 开头
awk '/^Tom/' file           # 整行以Tom开头
awk '$1 ~ /^T/' file        # 第1列以T开头

# 结尾
awk '/th$/' file            # 整行以th结尾
awk '$1 ~ /th$/' file       # 第1列以th结尾

# 字符类
awk '/[0-9]/' file          # 包含数字
awk '/[a-zA-Z]/' file       # 包含字母
awk '/^[0-9]+$/' file       # 纯数字

# 否定
awk '!/Math/' file          # 不包含Math
awk '$1 !~ /Math/' file     # 第1列不包含Math
```

### 5.4 范围匹配

```bash
# 行号范围
awk 'NR >= 5 && NR <= 10' file     # 第5到10行
awk 'NR == 5, NR == 10' file       # 从第5行到第10行

# 模式范围
awk '/start/, /end/' file         # 从包含start的行到包含end的行
awk 'NR==3, NR==5' file           # 第3到5行
```

---

## 6. 格式化输出

### 6.1 printf 语法

```bash
awk '{ printf "format", args }' file
```

### 6.2 格式符

| 格式 | 说明 | 示例 |
|------|------|------|
| `%s` | 字符串 | `"Tom"` |
| `%d` | 整数 | `95` |
| `%f` | 浮点数 | `95.5` |
| `%e` | 科学计数法 | `9.55e+1` |
| `%c` | 字符 | `A` |
| `%o` | 八进制 | `157` |
| `%x` | 十六进制 | `5f` |
| `%%` | 转义百分号 | `%` |

### 6.3 宽度修饰

| 修饰 | 说明 | 示例 |
|------|------|------|
| `%10s` | 右对齐，占10字符 | `     Tom` |
| `%-10s` | 左对齐，占10字符 | `Tom     ` |
| `%.3s` | 截取前3字符 | `Tom` |
| `%10.3s` | 右对齐，截取3字符 | `     Tom` |
| `%-10.3s` | 左对齐，截取3字符 | `Tom      ` |
| `%05d` | 补零 | `00095` |

### 6.4 示例

```bash
# 表格式输出
awk 'BEGIN { 
    printf "%-10s %-5s %s\n", "姓名", "分数", "科目"
    printf "%-10s %-5s %s\n", "----", "----", "----"
}
{ printf "%-10s %-5d %s\n", $1, $2, $3 }' score.txt

# 输出：
# 姓名         分数 科目
# ----         ---- ----
# Tom          95   Math
# Alice        88   English
```

---

## 7. 运算与统计

### 7.1 算术运算

```bash
# 基本运算
awk '{ print $2 + 1 }' file     # 加
awk '{ print $2 - 1 }' file     # 减
awk '{ print $2 * 2 }' file     # 乘
awk '{ print $2 / 2 }' file     # 除
awk '{ print $2 % 2 }' file     # 取余
awk '{ print $2 ^ 2 }' file     # 幂运算
```

### 7.2 累加/累乘

```bash
# 累加
awk '{ sum += $2 } END { print sum }' file

# 累乘
awk '{ prod *= $2 } END { print prod }' file

# 递减
awk 'BEGIN { count = 5 } { count-- } END { print count }' file
```

### 7.3 统计示例

```bash
# 求和
awk '{ sum += $2 } END { print sum }' file

# 平均值
awk '{ sum += $2 } END { print sum/NR }' file

# 最大值
awk 'BEGIN { max = 0 } $2 > max { max = $2 } END { print max }' file

# 最小值
awk 'BEGIN { min = 999999 } $2 < min { min = $2 } END { print min }' file

# 计数（条件）
awk '$2 >= 60 { pass++ } END { print pass }' file

# 方差
awk '{ 
    sum += $2; 
    sumsq += $2*$2 
} END { 
    print sqrt(sumsq/NR - (sum/NR)^2) 
}' file
```

---

## 8. 数组

### 8.1 基本数组

AWK 只支持**关联数组**（类似 Python 的字典）：

```bash
# 定义数组
awk 'BEGIN {
    arr["name"] = "Tom"
    arr["age"] = 25
    print arr["name"]
}'

# 数组遍历
awk 'END {
    for (key in arr) {
        print key, arr[key]
    }
}' file
```

### 8.2 数组用作计数器

```bash
# 统计每个值出现的次数
awk '{ count[$3]++ } END { 
    for (k in count) 
        print k, count[k] 
}' file

# 示例数据：
# Tom Math
# Alice Math
# Bob English

# 输出：
# Math 2
# English 1
```

### 8.3 数组用作查找表

```bash
# 第一遍建立查找表，第二遍使用
awk 'NR==FNR { 
    rate[$1] = $2   # 建立汇率表: USD=7.0
    next 
} { 
    printf "%s = %.2f CNY\n", $1, $1 * rate[$1] 
}' rate.txt goods.txt
```

### 8.4 数组函数

```bash
# 判断key是否存在
awk '{
    if ("key" in arr) {
        print "exists"
    }
}' file

# 删除元素
awk 'END { delete arr["key"] }' file

# 删除整个数组
awk 'END { delete arr }' file

# 数组长度
awk 'BEGIN { print length(arr) }'

# 获取数组索引
awk 'BEGIN {
    arr[1] = "a"
    arr[2] = "b"
    n = asort(arr)  # 排序，返回数量
    print n
}'
```

---

## 9. 字符串函数

### 9.1 常用函数表

| 函数 | 说明 | 示例 |
|------|------|------|
| `length(s)` | 字符串长度 | `length("abc") → 3` |
| `toupper(s)` | 转大写 | `toupper("abc") → ABC` |
| `tolower(s)` | 转小写 | `tolower("ABC") → abc` |
| `substr(s, start, len)` | 截取子串 | `substr("abc", 2, 2) → bc` |
| `index(s, sub)` | 查找位置 | `index("abc", "b") → 2` |
| `match(s, r)` | 正则匹配 | `match("abc", /b/) → 2` |
| `gsub(r, repl, s)` | 全局替换 | `gsub(/a/, "x", "aaa") → xxx` |
| `sub(r, repl, s)` | 单次替换 | `sub(/a/, "x", "aaa") → xaa` |
| `split(s, arr, sep)` | 分割数组 | `split("a,b,c", arr, ",")` |
| `sprintf(fmt, ...)` | 格式化 | `sprintf("%s-%d", "a", 1)` |

### 9.2 示例

```bash
# 字符串长度
echo "Hello" | awk '{ print length($0) }'
# 输出: 5

# 转大写
echo "hello" | awk '{ print toupper($0) }'
# 输出: HELLO

# 截取
echo "Hello" | awk '{ print substr($0, 2, 3) }'
# 输出: ell

# 替换
echo "aabbcc" | awk '{ gsub(/b/, "X"); print }'
# 输出: aaXXcc

# 分割
echo "a,b,c" | awk '{ n=split($0, arr, ","); print arr[1], arr[3] }'
# 输出: a c
```

---

## 10. 控制流程

### 10.1 if 语句

```bash
awk '{
    if ($2 >= 90) {
        grade = "A"
    } else if ($2 >= 80) {
        grade = "B"
    } else if ($2 >= 60) {
        grade = "C"
    } else {
        grade = "D"
    }
    print $1, grade
}' file
```

### 10.2 for 循环

```bash
# 数组遍历
awk '{
    for (key in arr) {
        print key, arr[key]
    }
}' file

# 数值循环
awk 'BEGIN {
    for (i = 1; i <= 5; i++) {
        print i
    }
}'
```

### 10.3 while 循环

```bash
awk '{
    i = 1
    while (i <= NF) {
        print $i
        i++
    }
}' file
```

### 10.4 do-while 循环

```bash
awk 'BEGIN {
    i = 1
    do {
        print i
        i++
    } while (i <= 5)
}'
```

### 10.5 三元运算符

```bash
# 简洁的条件表达式
awk '{ print ($2 >= 60 ? "及格" : "不及格") }' file
```

### 10.6 break 和 continue

```bash
# break - 跳出循环
awk '{
    for (i = 1; i <= 10; i++) {
        if (i == 5) break
        print i
    }
}'

# continue - 跳过本次循环
awk '{
    for (i = 1; i <= 5; i++) {
        if (i == 3) continue
        print i
    }
}'
```

---

## 11. 分隔符

### 11.1 输入字段分隔符 (FS)

```bash
# 方法1: -F 简写
awk -F',' '{ print $1 }' file.csv

# 方法2: FS 变量
awk 'BEGIN { FS = "," } { print $1 }' file.csv

# 方法3: 正则分隔符
awk 'BEGIN { FS = "[,:]" } { print $1 }' file

# 方法4: 固定宽度
awk 'BEGIN { FIELDWIDTHS = "3 4 5" } { print $1, $2, $3 }' file
```

### 11.2 输出字段分隔符 (OFS)

```bash
# 设置输出分隔符
awk 'BEGIN { OFS = " | " } { print $1, $2 }' file
# 输出: Tom | 95

# 保持分隔符一致
awk 'BEGIN { FS = ","; OFS = " - " } { print $1, $2 }' file.csv
```

### 11.3 记录分隔符 (RS)

```bash
# 按段落读取（空行分隔）
awk 'BEGIN { RS = "" } { print NR ": " $0 }' file

# 按特定字符分隔
awk 'BEGIN { RS = ";" } { print }' file.txt

# 保持多行记录
awk '{ printf "%s%s", $0, (RT ~ /;/ ? "\n" : " ") }' file
```

### 11.4 输出记录分隔符 (ORS)

```bash
# 双行分隔
awk 'BEGIN { ORS = "\n\n" } { print }' file

# 紧凑输出
awk 'BEGIN { ORS = "" } { print }' file
```

---

## 12. 多文件处理

### 12.1 处理多个文件

```bash
# 逐文件处理
awk '{ print FILENAME, $0 }' file1.txt file2.txt

# 文件名不同时重置 FNR
awk 'BEGIN { FILENAME != prev { FNR = 0; prev = FILENAME } 
    { print FNR, $0 }
}' file1.txt file2.txt
```

### 12.2 文件间关联

```bash
# 合并两个文件
awk 'NR==FNR { a[NR]=$0; next } { print a[FNR], $0 }' file1.txt file2.txt

# 左连接
awk 'NR==FNR { a[$1]=$2; next } { print $1, a[$1] }' lookup.txt data.txt
```

### 12.3 getline

```bash
# 读取下一行（不参与主循环）
awk '{
    print $1
    getline
    print $1
}' file

# 从文件读取
awk '{
    while ((getline line < "other.txt") > 0) {
        print line
    }
}' file
```

---

## 13. 函数

### 13.1 内置函数

```bash
# 数学函数
awk 'BEGIN {
    print sqrt(16)       # 平方根: 4
    print exp(1)         # e的x次方: 2.71828
    print log(2)         # 自然对数: 0.693147
    print sin(0)         # 正弦: 0
    print cos(0)        # 余弦: 1
    print int(3.7)      # 取整: 3
    print rand()        # 随机数: 0-1
    srand(123)          # 随机种子
}'
```

### 13.2 自定义函数

```bash
awk '
function max(a, b) {
    return (a > b) ? a : b
}
function min(a, b) {
    return (a < b) ? a : b
}
{ 
    print "max:", max($1, $2)
    print "min:", min($1, $2)
}' file
```

### 13.3 递归函数

```bash
awk '
function fact(n) {
    if (n <= 1) return 1
    return n * fact(n-1)
}
{ 
    print fact($1) 
}' <<< "5"
# 输出: 120
```

---

## 14. 实战案例

### 14.1 日志分析

```bash
# 访问日志格式: IP 时间 请求 状态 字节
# 192.168.1.1 - - [10/Oct/2024:13:55:36] GET /index 200 2326

# 提取IP
awk '{ print $1 }' access.log

# 统计每个IP访问次数
awk '{ ip[$1]++ } END { for (i in ip) print i, ip[i] }' access.log | sort -rn

# 找出404错误
awk '$9 == 404 { print $7 }' access.log

# 统计状态码
awk '{ code[$9]++ } END { for (c in code) print c, code[c] }' access.log

# 计算总流量
awk '{ bytes += $10 } END { print bytes }' access.log

# 找出访问量最大的IP
awk '{ ip[$1]++ } END { 
    for (i in ip) 
        if (ip[i] > max) { max = ip[i]; top = i }
    print top, max
}' access.log
```

### 14.2 CSV 处理

```bash
# CSV文件: name,age,city,score

# 提取某列
awk -F',' '{ print $1 }' data.csv

# 过滤行
awk -F',' '$4 > 90' data.csv

# 计算列总和
awk -F',' '{ sum += $4 } END { print sum }' data.csv

# 去重
awk -F',' '!seen[$1]++' data.csv

# 多列排序
awk -F',' '{ print $0 }' data.csv | sort -t',' -k3 -n
```

### 14.3 文本转换

```bash
# CSV 转 JSON
awk -F',' 'BEGIN { print "[" } 
{ 
    printf "  {\"name\":\"%s\",\"age\":\"%s\",\"city\":\"%s\"}", $1, $2, $3
    if (NR < 10) print ","; else print ""
} 
END { print "]" }' data.csv

# 文本列对齐
awk '{ printf "%-20s %-10s\n", $1, $2 }' file

# 数字格式化
awk '{ printf "%s: %.2f\n", $1, $2 }' file
```

### 14.4 数据清洗

```bash
# 去除首尾空格
awk '{ gsub(/^[ \t]+|[ \t]+$/, ""); print }' file

# 去除空行
awk 'NF > 0' file

# 去除重复行
awk '!seen[$0]++' file

# 奇偶行处理
awk 'NR%2==1' file   # 奇数行
awk 'NR%2==0' file   # 偶数行

# 替换文本
awk '{ gsub(/old/, "new"); print }' file
```

---

## 15. 性能优化

### 15.1 技巧

| 技巧 | 说明 |
|------|------|
| 减少 I/O | 使用 printf 而非多次 print |
| 减少正则 | 条件判断比正则快 |
| 提前过滤 | 先用条件过滤，减少处理 |
| 关闭管道 | 用完 close() |
| 简化数组 | 避免嵌套数组 |
| 使用 next | 提前跳过无需处理的行 |

### 15.2 示例

```bash
# 优化前
awk '{ if ($1 ~ /error/) { print } }' file

# 优化后
awk '/error/ { print }' file

# 优化前
awk '{ for (i=1; i<=NF; i++) { if ($i > 0) print $i } }' file

# 优化后
awk '{ for (i=1; i<=NF; i++) $i > 0 && print $i }' file
```

---

## 16. 常见问题

### 16.1 错误解决

| 问题 | 原因 | 解决 |
|------|------|------|
| 无输出 | 文件路径错误 | 检查文件是否存在 |
| 列数不对 | 分隔符错误 | 用 `-F` 指定正确分隔符 |
| 全部输出 | 条件写法错误 | 检查 `$1`、`$2` |
| END不执行 | 输入为空 | 确认文件有内容 |
| 数组为空 | 作用域问题 | 在 BEGIN 外定义 |

### 16.2 调试技巧

```bash
# 查看每行处理
awk '{ print NR, $0 }' file

# 查看字段分割
awk -F',' '{ print NF, $1, $2 }' file

# 调试模式
awk --dump-variables='' file
awk --profile file
```

---

## 17. 速查表

### 17.1 变量速查

| 变量 | 说明 |
|------|------|
| `$0` | 整行 |
| `$1~$N` | 第N列 |
| `$NF` | 最后1列 |
| `NR` | 行号 |
| `NF` | 字段数 |
| `FS` | 输入分隔符 |
| `OFS` | 输出分隔符 |

### 17.2 条件速查

```bash
$2 > 90           # 数字比较
$1 == "Tom"       # 字符串相等
/Math/            # 正则匹配
$2 ~ /[0-9]/      # 字段正则
NR==1             # 行号
```

### 17.3 函数速查

```bash
print             # 打印
printf            # 格式化打印
length()          # 长度
toupper()         # 转大写
tolower()         # 转小写
substr()          # 截取
gsub()            # 全局替换
split()           # 分割
```

### 17.4 常用命令

```bash
# 基础
awk '{ print }'           # 打印整行
awk '{ print $1 }'        # 打印第1列

# 条件
awk '$2 > 90'             # 过滤

# 计算
awk '{ sum += $2 } END { print sum }'  # 求和

# 统计
awk '{ count[$1]++ } END { for(k in count) print k, count[k] }'

# 格式
awk '{ printf "%s %d\n", $1, $2 }'      # 格式化
```

---

## 附录：AWK vs 其他工具

| 工具 | 特点 |
|------|------|
| AWK | 轻量、强大、适合小任务 |
| Sed | 流编辑器，适合替换 |
| Grep | 文本搜索 |
| Cut | 简单列提取 |
| Sort | 排序 |
| Uniq | 去重/计数 |
| Perl | 更强大的正则和文本处理 |
| Python | 全能编程语言 |

---
