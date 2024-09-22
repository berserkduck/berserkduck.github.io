---
title: MySQL递归
date: 2024-09-22T20:37:31+08:00
lastmod: [":fileModTime", "lastmod"]
author: Ryan
summary: MySQL递归公共表表达式简介
draft: false
categories:
  - MySQL
---
---
## 递归
递归是一种程序调用自身的编程思想，通常用于解决那些可以分解为更小同类问题的任务，比如计算阶乘、求斐波那契数列，遍历树结构等。

## 回顾两个数学概念
### 阶乘
一个正整数的阶乘（factorial）是所有小于及等于该数的正整数的积，并且0的阶乘为1。自然数n的阶乘写作n!

$$
n!=1×2×3×...×(n-1)×n
$$

在Python中求n的阶乘
```python
def factorial(n):  
    if n == 0:  
        return 1  
    else:  
        return n * factorial(n-1)  

print(factorial(5)) # 输出120，即1×2×3×4×5=120
```

### 斐波那契数列
斐波那契数列（Fibonacci series）是指这样一个数列：0，1，1，2，3，5，8，13，21，34，55，89……这个数列从第3项开始 ，每一项都等于前两项之和，表达式为：
$$
F[n]=F[n-1]+F[n-2] (n>1, F[0]=0, F[1]=1)
$$

在Python中求斐波那契数（这里求第六个斐波那契数）
```python
def fibonacci(n):  
    if n == 0:  
        return 0  
    elif n == 1:  
        return 1  
    else:  
        return fibonacci(n - 1) + fibonacci(n - 2)  
  
print(fibonacci(6))  # 输出 8
```
> 要注意，第0个斐波那契数为0，有些资料中省略了0
## MySQL递归查询
MySQL 8.0 引入了递归公共表表达式（[Recursive Common Table Expressions](https://dev.mysql.com/doc/refman/8.0/en/with.html#common-table-expressions-recursive)），可以用来处理具有层级关系的数据，比如组织结构图、文件系统目录等。
### 结构
与普通CTE的区别是递归CTE用`WITH RECURSIVE`引出，基本语法如下
```sql
WITH RECURSIVE cte_name (column1, column2, ...) AS
( 
	初始化部分 
	UNION ALL 
	递归部分 where 条件
)
SELECT ... FROM cte_name;
```
- *cte_name*：临时表
- *column1, column2, ...*：临时表中的列，用逗号间隔
- *初始化部分*：通常是一个简单的查询，返回递归的初始行。
- *递归部分*：引用了CTE本身来生成后续的行，直到满足终止条件

举一个例子，比如要生成数字序列1-3，可以这样查询：

```sql
WITH RECURSIVE cte (n) AS
( 
	SELECT 1 
	UNION ALL 
	SELECT n + 1 FROM cte WHERE n < 3
) 
	SELECT * FROM cte;
```
输出：

| n   |
| :-- |
| 1   |
| 2   |
| 3   |


这段代码是这样运行的：

1. 执行初始化部分：`select 1`，得到`n = 1`，也就是下面这个表（cte）

| n   |
| :-- |
| 1   |


2. 执行递归部分：`SELECT 1 + 1 FROM cte WHERE n < 3`，得到`n = 2`
3. 执行union all，得到下表

| n   |
| :-- |
| 1   |
| 2   |

4. 执行递归部分：`SELECT 2 + 1 FROM cte WHERE n < 3`，得到n = 3
5. 执行union all，得到下表

| n   |
| :-- |
| 1   |
| 2   |
| 3   |

6. 执行递归部分，条件`3 < 3 `不满足，跳出递归
7. 执行`SELECT * FROM cte`，输出最终结果
### 阶乘
用递归查询计算阶乘，这里定义临时表为Factorial，有两列：n和result，最终支取result的值
```sql
WITH RECURSIVE Factorial(n, result) AS
(  
    SELECT 1, 1  
    UNION ALL  
    SELECT n + 1, result * (n + 1)  
    FROM Factorial  
    WHERE n < 5  
)  
SELECT result FROM Factorial WHERE n = 5;
```

| result |
| :--- |
| 120 |

### 斐波那契数列
这是MySQL官方文档中的案例，n可以理解成序号，fib_n表示第n个斐波那契数，next_fib_n表示下一个斐波那契数
```sql
WITH RECURSIVE fibonacci (n, fib_n, next_fib_n) AS
(
  SELECT 1, 0, 1
  UNION ALL
  SELECT n + 1, next_fib_n, fib_n + next_fib_n
    FROM fibonacci WHERE n < 10
)
SELECT * FROM fibonacci;
```

| n   | fib\_n | next\_fib\_n |
| :-- | :----- | :----------- |
| 1   | 0      | 1            |
| 2   | 1      | 1            |
| 3   | 1      | 2            |
| 4   | 2      | 3            |
| 5   | 3      | 5            |
| 6   | 5      | 8            |
| 7   | 8      | 13           |
| 8   | 13     | 21           |
| 9   | 21     | 34           |
| 10  | 34     | 55           |

