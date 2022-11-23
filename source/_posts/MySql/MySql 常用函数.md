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
- LEFT(str, length) 从左开始截取指定长度的子字符串
```sql
# 从左边截取长度为2的子字符串，返回 HE
SELECT LEFT("HELLO", 2)
```
- RIGHT(str, length) 从右边开始截取指定长度的子字符串
```sql
# 返回 LO
SELECT RIGHT("HELLO", 2)
```
- TRIM([{both | leading | trailing} [remstr] form] str) 将字符串 str去除 remstr 所指定的前缀或后缀，返回结果字符串。如果没有指定标识符both、leading，或trailing，则默认采用 both，即将前后缀都删除。remstr 其实是个可选参数，如果没有指定它，则删除的是空格
```sql
# 去掉字符串前后的空格，注意部分：制表符号空白是去不掉的
SELECT TRIM(" HELLO ")
# 等价于
SELECT TRIM(BOTH ' ' FROM  ' HELLO ')

# 去掉指定字符前缀
SELECT TRIM(LEADING 'hello' FROM 'helloWorld');
# 去掉指定字符后缀
SELECT TRIM( TRAILING, 'hello' FROM 'Worldhello');
# 去掉指定字符前后缀
SELECT TRIM(BOTH 'Hello' FROM  'HelloWorldHello');
```