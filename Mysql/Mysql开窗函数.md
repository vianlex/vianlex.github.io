# MySql 开窗函数

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
开窗函数语法格式：`func_name([<column>]) OVER([PARTITION BY <column> | window_name] [ORDER BY <column>] [[ROWS | RANGE] BETWEEN frame_start AND frame_end])`
 - OVER 函数中如果 paritition_clause 或者 window_name，则开窗函数统计分析全部数据
 - PARTITION BY 子句：与 GROUP BY 类似，用于按照指定字段进行分组数据 
 - ORDER BY 子句：按照指定字段排序 partition by 子句分组的数据，如果没有指定 partition by 则排序全部数据
 - [ROWS | RANGE] BETWEEN frame_start AND frame_end 子句：在 PARTITION 分区的基础，再划分子分区
    - ROWS: ROWS 划定的子分区是以当前行为基准，前后偏移行数的范围
    - RANGE: 根据当前行的值划分子分区，注意如果使用 expr PRECEDING/FOLLOWING 必须要 ORDER BY 一个数字或者时间类型的字段
    - 子分区 frame_start(起始边界行)、frame_end(结束边界行) 可选值如下：
        - CURRENT ROW：对于 ROWS 而言，以当前行作为边界行，对于 RANGE 而言，与当前行的值相同的都作为边界行
        - UNBOUNDED PRECEDING：以 PARTITION 分区的第一行作为起始边界行
        - UNBOUNDED FOLLOWING：以 PARTITION 分区的最后一行作为终点边界行
        - expr PRECEDING：对于 ROWS 而言 expr 值只能为数字，边界行是当前行的前前几行，对于 RANGE 而言 expr 值可以是数字或者时间表达式，行值等于大于ORDER BY 字段当前行的值减去表达式的值所得值的即为边界行
        - expr FOLLOWING：FOLLOWING 根据 PRECEDING 相对的，对于 ROWS 而言边界行是当前行的后几行，对于 RANGE 而言，行值等于小于 ORDER BY 字段当前行的值加上 expr 表达式的值即为边界行

## 例子说明

1. 语法例子

```sql
/** PARTITION BY 直接写在 OVER 函数中  **/
SELECT *, ROW_NUMBER() OVER(PARTITION BY `month` ORDER BY salary) AS `rownum`
FROM (
SELECT '2月' AS `month`, 3000 AS salary
UNION ALL 
SELECT '1月' AS `month`, 4000 AS salary
) t

/** 使用 WINDOW 命名方式定义 PARTITION BY 字句和 ORDER BY 字句 **/
SELECT `month`, `salary`,
   ROW_NUMBER() OVER(w1 ORDER BY salary) AS `rownum1`,
   ROW_NUMBER() OVER(w2 ORDER BY salary) AS `rownum2`,
   ROW_NUMBER() OVER w3 AS `rownum3`,
   ROW_NUMBER() OVER w4 AS `rownum4`
FROM (
SELECT '2月' AS `month`, 3000 AS salary
UNION ALL 
SELECT '1月' AS `month`, 4000 AS salary
) t WINDOW w1 AS (), w2 AS (PARTITION BY `month`), w3 AS ( ORDER BY salary), w4 AS (PARTITION BY `month` ORDER BY salary)
```

1. ROW_NUMBER、RANK、DENSE_RANK 排序区别

```sql
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

3. range 子分区窗口使用说明

```sql
/* 注意 range 是 parition by 分区下的子分区，如果没有 paritiion by 则默认是所有记录下的子分析，使用 expr following 必须要 order by 时间或者数字类型的字段 */
SELECT 
   `name`,`month`, `salary`,
   /*  在 month 分区下，计算子分区为(当前行month值 <= month <= 当前行month值 + 1)内的行 */
   SUM(salary) OVER(PARTITION BY month ORDER BY month RANGE BETWEEN CURRENT ROW AND 1 FOLLOWING) orderMonth,
   /*  在 month 分区下，计算子分区(当前行salary的值 <= salary <= 当前行salary值+100  )内的行 */
   SUM(salary) OVER(PARTITION BY month ORDER BY salary RANGE BETWEEN CURRENT ROW AND 100 FOLLOWING) orderSalary,
   /*  没有 parition by 则表示，在所有记录的分区下，计算子分区(当前行month值 <= month <= 当前行month值 + 1)内的行 */
	SUM(salary) OVER(ORDER BY month RANGE BETWEEN CURRENT ROW AND 1 FOLLOWING)
FROM (
SELECT '李思' AS `name`, 300 AS salary, 1 AS `month`
UNION ALL 
SELECT '张三' AS `name`, 300 AS salary, 1 AS `month`
UNION ALL
SELECT '李思' AS `name`, 300 AS salary, 2 AS `month`
UNION ALL 
SELECT '张三' AS `name`, 350 AS salary, 2 AS `month`
UNION ALL 
SELECT '小鱼' AS `name`, 300 AS salary, 1 AS `month`
UNION ALL 
SELECT '小鱼' AS `name`, 400 AS salary, 2 AS `month`
UNION ALL
SELECT '小明' AS `name`, 300 AS salary, 1 AS `month`
UNION ALL 
SELECT '小明' AS `name`, 450 AS salary, 2 AS `month`
) t
```

4. rows 子分区窗口使用说明

```sql
SELECT 
   `month`, `salary`,
   ROW_NUMBER() OVER(),
   SUM(salary) OVER(ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING),
   /*在 month 分区下，计算的子分区的范围是(当前行的row_number  <= row_number <= 当前行的row_number + 1)之间的行*/
   SUM(salary) OVER(PARTITION BY `month` ORDER BY `month` ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING),
   ROW_NUMBER() OVER(PARTITION BY `month` ORDER BY `month` )

FROM (
SELECT '李思' AS `name`, 300 AS salary, '1月份' AS `month`
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
1、https://dev.mysql.com/doc/refman/8.0/en/window-functions.html