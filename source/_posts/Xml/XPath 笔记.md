---
title: XPath 笔记
---

## XPath 节点术语
在 XPath 中，有七种类型的节点：元素、属性、文本、命名空间、处理指令、注释以及文档（根）节点。XML 文档是被作为节点树来对待的。树的根被称为文档节点或者根节点。

## 节点关系
- 父（Parent）， 每个元素以及属性都有一个父节点。
- 子（Children）, 元素节点可有零个、一个或多个子节点。
- 同胞（Sibling），拥有相同的父的节点。
- 先辈（Ancestor），某节点的父节点、及父父节点，等等。
- 后代（Descendant），某个节点的子节点，及子子节点，等等。

## XPath 语法
示例语法 XML 实例文档，如下：
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<bookstore>
    <book>
        <title lang="eng">Harry Potter</title>
        <price>29.99</price>
    </book>
    <book>
        <title lang="eng">Learning XML</title>
        <price>39.95</price>
    </book>
</bookstore>
```
### 选取节点

|                  |                    |
| ---------------- | ------------------ | 
| 表达式	 |  描述 | 
| nodename	| 选取此节点的所有子节点。|
| /	        | 表示文档树的起始 |
| //	    | 匹配文档中的任何节点，而不考虑它们的位置。|
| .	        | 选取当前节点。|
| ..	    | 选取当前节点的父节点。|
| @	        | 选取属性。|

使用例子：
```bash
# 获取文档中的 bookstore 节点
/bookstore

# 获取文档 bookstore 节点下的 book 节点
/bookstore/book

# 获取文档中的所有 book 节点
//book

# 获取文档中的所有 title 节点 
//title

# 获取名为 lang 的所有属性
//@lang

```

## 谓语选择
谓语用来查找某个特定的节点或者包含某个指定的值的节点，被嵌在方括号中使用。

|       |       | 
| ----- | ------|
| /bookstore/book[1]	| 选取属于 bookstore 子元素的第一个 book 元素。|
| /bookstore/book[last()]	| 选取属于 bookstore 子元素的最后一个 book 元素。|
| /bookstore/book[last()-1]	| 选取属于 bookstore 子元素的倒数第二个 book 元素。|
| /bookstore/book[position()<3]	| 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。|
| //title[@lang]	| 选取所有拥有名为 lang 的属性的 title 元素。|
| //title[@lang='eng']	| 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。|
| /bookstore/book[price>35.00]	| 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。|
| /bookstore/book[price>35.00]/title	| 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。|


## 选取未知节点
XPath 通配符可用来选取未知的 XML 元素。
|   |   |
|  -----  | ----- |
| 通配符	| 描述 |
| *	 | 匹配任何元素节点。如 //book/*，选取 book 节点下面的所有节点，如 `//*`，选择文档中的所有元素。|
| @*	| 匹配任何属性节点。如 //title[@*] 选取所有带有属性的 title 元素 |
| node() |	匹配任何类型的节点。如 /bookstore/node() 选取 bookstore 节点下的任何节点 |


## 选取若干路径
通过在路径表达式中使用“|”运算符，您可以选取若干个路径。

|  |    |
|  --- | --- |
路径表达式	结果
//book/title | //book/price	选取 book 元素的所有 title 和 price 元素。
//title | //price	选取文档中的所有 title 和 price 元素。
/bookstore/book/title | //price	选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。

## XPath 轴
轴可定义相对于当前节点的节点集。

|   |   |
|  --- | --- |
| 轴名称	 | 结果 |
ancestor	选取当前节点的所有先辈（父、祖父等）。
ancestor-or-self	选取当前节点的所有先辈（父、祖父等）以及当前节点本身。
attribute	选取当前节点的所有属性。
child	选取当前节点的所有子元素。
descendant	选取当前节点的所有后代元素（子、孙等）。
descendant-or-self	选取当前节点的所有后代元素（子、孙等）以及当前节点本身。
following	选取文档中当前节点的结束标签之后的所有节点。
namespace	选取当前节点的所有命名空间节点。
parent	选取当前节点的父节点。
preceding	选取文档中当前节点的开始标签之前的所有节点。
preceding-sibling	选取当前节点之前的所有同级节点。
self	选取当前节点。




## 参考链接
1. https://www.w3school.com.cn/xpath/index.asp
2. [在线测试地址](https://extendsclass.com/xpath-tester.html#sample)