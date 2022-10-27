---
title: MySql 开窗函数
---
## 简介
MySql 是从8.0版本之后开始支持开窗函数，开窗函数也叫分析函数。


## 窗口函数
除了聚合函数可以作为开窗函数， MySql 还提供了以下开窗函数。用于解决复杂报表统计需求的功能强大的函数。
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
 
## 窗口函数语法
开窗函数语法格式：`func_name([<字段名>]) over([partition by <分组字段>] [order by <排序字段>] [<window>]) `
 - over 函数中不包含 paritition by 子句，则开窗函数统计分析全部数据
 - partition by 子句：可以看作与 GROUP BY 相同，按照指定字段进行分组数据
 - order by 子句：按照指定字段排序 partition by 子句分组的数据，如果没有指定 partition by 则排序全部数据
 - window 子句：用于 order by 排序后指定起始行和结束行范围内的数据统计分析。使用 window 子句必须先排序，其语法格式：`rows | range between start_expr and end_expr`
    - rows 和 range 区别是，rows 是按 order by 排序后的实际行号来计算行数范围的，range 会把 partition by 分组的字段值相同的看是同一个行
    - start_expr 可选值如下：
        - unbounded preceding：over 分区排序后的第一行作为窗口的起始行
        - current row：以当前行作为窗口的起始行
        - n preceding：以当前行的前面第 n(注意：n 是一个具体的数字) 行作为窗口的起始行
        - n following：以当前行的后面第 n 行作为窗口的起始行
    - end_expr 可选值如下：
        - unbounded following：over 分区排序后的最后一行作为窗口终点
        - current row：以当前行作为窗口终点
        - n preceding：以当前行的前面第 n 行作为窗口终点
        - n following：以当前行的后面第 n 行作为窗口终点

## 例子说明
1. ROW_NUMBER、RANK、DENSE_RANK 排序区别
```
SELECT salary,
	ROW_NUMBER() OVER(ORDER BY salary) AS `rown_umber`,
	RANK() OVER(ORDER BY salary ) AS `rank`,
	DENSE_RANK() OVER(ORDER BY salary) AS `dense_rank` 
FROM (
SELECT 1000 AS salary 
UNION ALL
SELECT 1000 AS salary 
UNION ALL
SELECT 2000 AS salary
UNION ALL
SELECT 3000 AS salary
UNION ALL 
SELECT 3000 AS salary
UNION ALL 
SELECT 4000 AS salary
) t
```
结果显示如下
| salary | rown_umber | rank | dense_rank | 
| ---: | ---: | ---: | ---: | 
| 1000 | 1 | 1 | 1 | 
| 1000 | 2 | 1 | 1 | 
| 2000 | 3 | 3 | 2 | 
| 3000 | 4 | 4 | 3 | 
| 3000 | 5 | 4 | 3 | 
| 4000 | 6 | 6 | 4 | 


2. rows 和 range 分段窗口区别
```
SELECT 
	`name`,`month`, `salary`,
	SUM(salary) OVER(PARTITION BY `month`) AS month_salary,
	SUM(salary) OVER(PARTITION BY `month` 
	 ORDER BY salary
	 rows between unbounded preceding AND 2 following )  AS rows_segment_salary,
     SUM(salary) OVER(PARTITION BY `month` 
	 ORDER BY salary
	 range between unbounded preceding AND 2 following )  AS range_segment_salary
FROM (
SELECT '李思' AS `name`, 200 AS salary, '1月份' AS `month`
UNION ALL 
SELECT '张三' AS `name`, 300 AS salary, '1月份' AS `month`
UNION ALL
SELECT '李思' AS `name`, 300 AS salary, '2月份' AS `month`
UNION ALL 
SELECT '张三' AS `name`, 350 AS salary, '2月份' AS `month`
UNION ALL 
SELECT '小鱼' AS `name`, 300 AS salary, '1月份' AS `month`
UNION ALL 
SELECT '小鱼' AS `name`, 400 AS salary, '2月份' AS `month`
UNION ALL
SELECT '小明' AS `name`, 300 AS salary, '1月份' AS `month`
UNION ALL 
SELECT '小明' AS `name`, 450 AS salary, '2月份' AS `month`
) t
```


## 参考链接
1、https://www.begtut.com/mysql/mysql-ntile-function.html
2、https://www.ngui.cc/51cto/show-543033.html?action=onClick
3、https://jishuin.proginn.com/p/763bfbd79587