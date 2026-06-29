# Python 学习笔记

## 变量的使用

Python 中的变量不需要声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。

### 变量的动态特性

Python 是动态类型语言，意味着一个变量可以在不同时间存储不同类型的数据。

```py
hello = "Hello World"
hello = 90
```

### 变量的赋值和引用

在 Python 中，变量实际上是对象的引用。当我们赋值变量时，Python 开辟内存创建对象并将变量指向该对象：

```py
a = "Hello World"
b = a # a 和 b 都引用了相同的对象
```

### 变量的作用域

```py

```

## 基础数据类型

### Number 数字类型

Python 支持 int、float、bool、complex（复数）。
 
```py
# int 类型
num = 90 
# 获取变量的类型
print(type(num))
# 判断变量是不是某个类型的实例
print(isinstance(num,int))

# float 类型
num = 90.99

# bool 类型
num = True
print(Flase == 0)
print(True==1)

# bool 类型是 int 的子类，所以可以数字类型相加
num = True + 2 # 等于 3
num = Flase + 2.2  # 等于 1.2 
```

## 数据类型转换

常用的类型转换函数如下：

| 函数	| 描述 | 
| -- | -- | 
|int()	| 将值转换为整数 |
|float()	| 将值转换为浮点数 |
|str()	| 将值转换为字符串 |
|bool()	| 将值转换为布尔值 |
|list()	| 将值转换为列表 |
|tuple() |	将值转换为元组 |
|set()  |	将值转换为集合 |
|dict() |	将值转换为字典 |

### 转整数

在 Python 中，可以将浮点数、布尔值或（数字、科学计数、二进制、十六进制）字符串转换为整数

```py
# 将浮点数转整数
print(12.01) # 输出 <class 'float'>
i = print(int(12.01)) # 输出 12

# 浮点数转整数
print(int(10.8))  # 输出: 10
print(int(-10.8)) # 输出: -10

# 布尔值转整数
print(int(True))  # 输出: 1
print(int(False)) # 输出: 0

# 字符串转整数
print(int("100")) # 输出: 100

# 二进制和十六进制的字符串转整数
print(int("1010", 2))  # 输出: 10
print(int("A", 16))    # 输出: 10
```

### 转浮点数

在 Python 中，可以将整数、布尔值或字符串（包含数字、科学计数的字符串，不支持二或者十六进制的字符串）转换为浮点数

```py
# 将整数转浮点数
print(type(12)) # 输出：<class 'int'>
f = print(float(12)) # 输出 12.0

# 布尔值转整数
print(float(True))  # 输出: 1.0
print(float(False)) # 输出: 0.0

# 字符串转整数
print(float("100")) # 输出: 100
```

### 转布尔类型

在 Python 中任何类型的对象都可以转换成布尔类型。

注意：在 Python 中`数字零（0, 0.0）、空字符串（""）、空列表（[]）、空元组（()）、空字典（{}）、None、False`零值和空值会视为 False，其他值则视为 True


### 转字符串类型

在 Python 中几乎所有的 Python 对象都可以转为字符串。

```py
# 布尔值转字符串
print(str(True)) # 输出: "True"

# 列表转字符串
print(str([1, 2, 3])) # 输出: "[1, 2, 3]"
```
