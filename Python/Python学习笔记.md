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

在 Python 中有四种不同级别的作用域：
- 局部作用域（Local Scope）：在函数内定义的变量
- 嵌套作用域（Enclosing Scope）：在外层函数中定义的变量
- 全局作用域（Global Scope）：在模块层次中定义的变量
- 内置作用域（Built-in Scope）：Python预定义的变量名

以上四种作用域构成了 Python 的 LEGB 规则：Local -> Enclosing -> Global -> Built-in。Python按照这个作用域的顺序查找变量。

#### 局部作用域（Local Scope）

局部作用域是在函数内部定义的变量，只在函数内部可见，函数执行结束后就会被销毁。

```py
def hello():
    x = 20
    print(f"hello count = {x}")
```

#### 全局作用域（Global Scope）

全局作用域是在模块（即.py文件）中最外层定义、函数外部定义的变量，在整个模块（文件）的任意位置可见。

```py
x = 20
def hello_global():
    print(f"hello count = {x}")
```

#### 嵌套作用域（Enclosing）

内层函数访问外层函数的局部变量，仅多层嵌套函数存在。

```py
def outer():
    x = 20  # 外层嵌套变量
    def inner():
        print(x)  # 向内找不到，去外层 outer 找（E）
    inner()

outer()  # 输出 20
```

#### 内置作用域（Built-in）

Python 解释器自带，所有模块（文件） / 函数，不需要引入，就能都能直接使用如：print, sum, max, int, str, list 等。

```py
# len 属于内置作用域
print(len([1,2]))
```

#### 关键字：global /nonlocal（修改上层变量）

##### global 关键字

对于全局变量，在函数内只读全局变量没问题，但是如果直接在函数内部修改全局变量，会被识别为新局部变量，报错。我们需要

```py
n = 10
def func1():
    # 可读
    print(n)
    # 未使用 global 关键字声明，直接修改会报错
    n = 20 
func1()

def func2():
    global n
    # global 声明后才能修改全局变量，否则会报错 
    n = 20
    print(n)

func2()
```

##### nonlocal 关键字

```py
def outer():
    val = 10
    def inner():
        # 声明为 nonlocal 变量，才能修改
        nonlocal val
        val = 99
    inner()
    print(val)  # 99

outer()
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
