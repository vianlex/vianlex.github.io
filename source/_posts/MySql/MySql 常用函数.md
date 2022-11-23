---
title: MySql 常用函数
---

## 1. 字符串函数
- SUBSTRING_INDEX(str, delimiter, index)  使用指定分割符(delimiter)，将字符串(str)分割成数组，并返回指定索引(index)的数组值
```sql
# 使用 ll 将字符串分割成数组，并返回数组第一个元素
SELECT SUBSTRING_INDEX("Hello","ll",1) 

# 根据逗号将字符串分割数组，并返回数组的最后一个值
SELECT SUBSTRING_INDEX("A,B,C,D",",",-1)
```
- SUBSTRING(str, pos[, length ]) 截取指定范围内的字符串，参数说明 pos 指定开始截取的位置，pos 可以取值正负，起始值是 1 或者 -1 ，负数表示从字符后面倒数，length 表示截取的长度，为空是默认截取到字符结尾
```sql
# 从第一个位置开始截取2个长度的子字符串，返回 HE
SELECT SUBSTRING("HELLO",1,2)

# 从倒数第二给位置开始，截取长度为 1 的子字符串，返回 L
SELECT SUBSTRING("HELLO", -2,1)

# 不指定 length 默认截取到字符串结尾，返回 LLO
SELECT SUBSTRING("HELLO",-3)
```
- REPLACE( str, from_str, to_str) 替换字符串
```sql
# 将逗号替换成空格
SELECT REPLACE("Hello,World",","," ")
```