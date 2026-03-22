# Awk 基础用法

--- 

## 一、基础概念

### 1.1 AWK 是什么？
- 文本处理工具，擅长处理结构化数据
- 名字来源：Aho + Weinberger + Kernighan
- 适用于：日志分析、CSV处理、数据统计

### 1.2 基本语法

```bash
awk 'pattern { action }' filename
```

### 1.3 数据处理流程

```bash 
┌─────────────────────────────────────────────────────────┐
│ AWK 工作流程 │
├─────────────────────────────────────────────────────────┤
│ │
│ 输入文件 │
│ │ │
│ ▼ │
│ ┌───────────────────┐ │
│ │ 读取一行 │ ◄── 每读一行执行一次 pattern │
│ │ (记录循环) │ 和 action │
│ └─────────┬─────────┘ │
│ │ │
│ ▼ │
│ ┌───────────────────┐ │
│ │ 条件匹配 pattern │ ◄── 模式/条件判断 │
│ │ (可选省略) │ 为真则执行 action │
│ └─────────┬─────────┘ │
│ │ │
│ ▼ │
│ ┌───────────────────┐ │
│ │ 执行 action │ ◄── 处理动作 │
│ │ (可选省略) │ 为空则默认打印整行 │
│ └─────────┬─────────┘ │
│ │ │
│ ┌──────┴──────┐ │
│ │ 下一行? │ │
│ └──────┬──────┘ │
│ │ │
│ ┌──────▼──────┐ │
│ │ END { } │ ◄── 所有行处理完后执行 │
│ └─────────────┘ │
│ │
└─────────────────────────────────────────────────────────┘
```

---

## 二、内置变量 

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `$0` | 当前整行 | `Tom 95 Math` |
| `$1` | 第1列 | `Tom` |
| `$2` | 第2列 | `95` |
| `$3` | 第3列 | `Math` |
| `$NF` | 最后1列 | `$3` |
| `$(NF-1)` | 倒数第2列 | `$2` |
| `NR` | 当前行号（多文件累计） | `1, 2, 3...` |
| `FNR` | 当前文件行号 | `1, 2, 3...` |
| `NF` | 当前行字段总数 | `3` |
| `FS` | 输入字段分隔符 | 空格/Tab |
| `OFS` | 输出字段分隔符 | 空格 |
| `RS` | 输入记录分隔符 | 换行 |
| `ORS` | 输出记录分隔符 | 换行 |
| `FILENAME` | 当前文件名 | `data.txt` |

---

## 三、使用示例

### 3.1 示例数据 

假设文件 `score.txt` 内容： 

```
Tom Math 95
Jerry English 88
Alice Math 92
Bob English 85
``` 
--- 

### 3.2 打印列

```bash
# 打印第1列
awk '{ print $1 }' score.txt # 打印第1列和第3列
awk '{ print $1, $3 }' score.txt # 自定义输出格式
awk '{ printf "%s: %d\n", $1, $3 }' score.txt
``` 
--- 

### 3.3 指定分隔符 

```bash
# CSV 文件，指定逗号为分隔符
awk -F',' '{ print $1, $2 }' data.csv # 或者用 FS 变量
awk 'BEGIN { FS="," } { print $1, $2 }' data.csv
``` --- ### 3. 条件过滤 ```bash
# 打印第3列 > 90 的行
awk '$3 > 90 { print $0 }' score.txt # 打印第1列 = "Tom" 的行
awk '$1 == "Tom" { print $0 }' score.txt # 包含关键词
awk '/Math/ { print $0 }' score.txt
``` --- ### 4. 运算与统计 ```bash
# 计算第3列总和
awk '{ sum += $3 } END { print sum }' score.txt # 计算平均值
awk '{ sum += $3; count++ } END { print sum/count }' score.txt # 求最大值
awk 'BEGIN { max=0 } $3 > max { max=$3 } END { print max }' score.txt
``` --- ### 5. BEGIN / END ```bash
# BEGIN: 处理前执行（常用于初始化）
awk 'BEGIN { print "成绩单" } { print $1, $3 } END { print "---" }' score.txt # 输出:
# 成绩单
# Tom 95
# Jerry 88
# Alice 92
# Bob 85
# ---
```
--- 

### 3.4 字符串函数 

```bash
# 转大写
awk '{ print toupper($1) }' score.txt # 拼接字符串
awk '{ print $1 "-" $2 }' score.txt # 截取子串
awk '{ print substr($1, 1, 3) }' score.txt # 正则替换
awk '{ gsub(/Tom/, "Tommy"); print }' score.txt
```
--- 

### 3.5 循环 

```bash
# 输出每列及其值
awk '{ for (i = 1; i <= NF; i++) { print "Column " i ": " $i }
}' score.txt
```

---

### 3.6 多文件处理 

```bash
# 处理多个文件
awk '{ print FILENAME, $0 }' file1.txt file2.txt # 连接两个文件
awk 'NR==FNR { a[NR]=$0 } NR>FNR { print a[NR-FNR], $0 }' file1.txt file2.txt
``` 
--- 

### 3.7 常用技巧 

```bash
# 去除首尾空格
awk '{ gsub(/^[ \t]+|[ \t]+$/, ""); print }' # 打印奇偶行
awk 'NR%2==1' file.txt # 奇数行
awk 'NR%2==0' file.txt # 偶数行 # 限制处理行数
awk 'NR<=10' file.txt # 前10行 # 从第N行开始
awk 'NR>=5' file.txt # 第5行开始 # 打印行号
awk '{ print NR, $0 }' file.txt
```
--- 

### 3.8 实用场景 

```bash
# 查看日志 IP 访问统计
awk '{ print $1 }' access.log | sort | uniq -c | sort -rn | head -10 # 提取 CSV 某列并求和
awk -F',' '{ sum += $4 } END { print sum }' data.csv # 格式化输出表头
awk 'BEGIN { printf "%-10s %-10s\n", "Name", "Score" } { printf "%-10s %-10s\n", $1, $3 }' score.txt
```
