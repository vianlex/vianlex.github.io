---
title: "Python 学习笔记"
linkTitle: "Python 学习笔记"
weight: 10
---
## 基础类型

|基础类型|说明|示例|类型判断|转换函数|
|---|---|---|---|---|
|`int`|整数，大小无限制|`v = 99`|`isinstance(v, int)`|`int(x)`|
|`float`|浮点数（小数）|`v = 3.14`|`isinstance(v, float)`|`float(x)`|
|`bool`|布尔；True/False，继承 int|`v = True`|`isinstance(v, bool)`|`bool(x)`|
|`complex`|复数|`v = 2+3j`|`isinstance(v, complex)`|`complex(x)`|
|`str`|字符串，引号包裹|`v = "python"`|`isinstance(v, str)`|`str(x)`|
|`list`|列表，可变 `[]`|`v = [1,2,3]`|`isinstance(v, list)`|`list(x)`|
|`tuple`|元组，不可变 `()`|`v = (1,2,3)`|`isinstance(v, tuple)`|`tuple(x)`|
|`range`|整数区间对象|`v = range(1,5)`|`isinstance(v, range)`|—（不可直接转换）|
|`set`|可变集合，自动去重|`v = {1,2,3}`|`isinstance(v, set)`|`set(x)`|
|`frozenset`|不可变集合|`v = frozenset({1,2})`|`isinstance(v, frozenset)`|`frozenset(x)`|
|`dict`|字典 key\-value|`v = {"a":1}`|`isinstance(v, dict)`|`dict()`|
|`NoneType`|空对象，唯一值 `None`|`v = None`|`isinstance(v, type(None))`|—|

### 可变 / 不可变类型说明

|分类|类型列表|特性|
|---|---|---|
|不可变|`int float bool str tuple frozenset`|修改值会生成新对象，原始对象不变|
|可变|`list dict set`|原地修改对象，内存 id 不变|

```python
hello = [1, 2,3]
# 查看对象的内存 id
print(id(hello))
```

### 列表（可变列表）

```python
# 初始化列表
ls = [1,2,3]
# 往列表中添加元素
ls.append(4)
ls.insert(0, 5)

```

### 元组（不可变）

```python
# 初始化
t1

```

### 类型转换示例

```python
# 数字转换
int("123")      # 123
float("2.5")    # 2.5
bool(0)         # False
bool("hello")   # True

# 序列、集合转换
str(1024)               # "1024"
list((1,2,3))           # [1,2,3]
tuple([10,20,30])       # (10,20,30)
set([1,1,2,2])          # {1, 2}
frozenset([1,2])        # frozenset({1,2})

# 字典构造
dict(name="张三", age=18)   # {'name': '张三', 'age': 18}
```

## 变量使用说明

Python 中的变量不需要声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。

### 变量的基本使用

Python 是动态类型语言，意味着一个变量可以在不同时间存储不同类型的数据。

```py
hello = "Hello World"
hello = 90
```

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
- 内置作用域（Built-in Scope）：Python 预定义的变量名

以上四种作用域构成了 Python 的 LEGB 规则：Local -> Enclosing -> Global -> Built-in。Python 按照这个作用域的顺序查找变量。

#### 局部作用域（Local Scope）

局部作用域是在函数内部定义的变量，只在函数内部可见，函数执行结束后就会被销毁。

```py
def hello():
    x = 20
    print(f"hello count = {x}")
```

#### 全局作用域（Global Scope）

全局作用域是在模块（即.py 文件）中最外层定义、函数外部定义的变量，在整个模块（文件）的任意位置可见。

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

#### 注意在 if/for/while 无独立作用域

在 if/for/while 语句块中它们不会创建局部作用域，它们内部定义的变量外部也可访问。只有 def / class / lambda 会生成独立作用域。

```py
if True:
    m = 5

# 正常输出 5
print(m)

for i in range(3):
    pass
# 正常输出 2
print(i)
```

## 类型注解

### 基础写法

```python
# 格式：变量名: 类型 = 值
a: int = 10
b: str = "hello"
c: float = 3.14
d: bool = True
e: list = [1, 2, 3]
f: dict = {"name": "tom"}
```

### 容器泛型注解

容器泛型注解在 `Python3.9+` 内置,

```python

```
