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
- SUBSTRING(str, start_pos[, end_pos ]) 截取指定范围内的字符串，索引说从 1 开始的，-1 表示最后一个元素的位置
```sql
# 截取字符串，返回 He
SELECT SUBSTRING("Hello",1,2)

# 截取最后一个字符
SELECT SUBSTRING("Hello", -1)
```
- REPLACE( str, from_str, to_str) 替换字符串
```sql
# 将逗号替换成空格
SELECT REPLACE("Hello,World",","," ")
```