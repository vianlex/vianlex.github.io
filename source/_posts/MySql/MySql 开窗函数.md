---
title: MySql 开窗函数
---
## 简介
MySql 是从8.0版本之后开始支持开窗函数，开窗函数也叫分析函数。


## 窗口函数
所有的聚合函数都可以作为开窗函数，除了聚合函数 MySql 还提供了以下开窗函数。
|  函数名   | 函数作用  |
|  ----  | ----  |
| ROW_NUMBER()  | 行号 |
| RANK()  | 排名 |
| DENSE_RANK() | 密集排名 |
| PRECENT_RANK() | 用于计算分区或结果集中行的百分位数|
| CUME_DIST() | 返回一组值中值的累积分布，表示值小于或等于行的值除以总行数的行数 |
| LAG() | 用于统计窗口内往上第 n 行值 |
| LEAD() | 用于统计窗口内往下第 n 行值 |
| FIRST_VALUE() | 选择窗口框架，分区或结果集的第一行 |
| LAST_VALUE() | 返回有序行集中的最后一行 |
| NTH_VALUE(expression, N) | 从有序行集中的第N行获取值 |
| NTILE() | 将排序分区中的行划分为特定数量的组，从每个组分配一个从一开始的桶号，对于每一行，NTILE()函数返回一个桶号，表示行所属的组 |
 





## 参考链接
1、https://www.begtut.com/mysql/mysql-ntile-function.html