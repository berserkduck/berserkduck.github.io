---
title: MySQL递归查询
date: 2024-09-22T20:37:31+08:00
lastmod: 2024-09-24T17:28:31+08:00
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
F[n]=F[n-1]+F[n-2]\\ (n>1, F[0]=0, F[1]=1)
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
{{< admonition type=note title="注意" open=true >}} 第0个斐波那契数为0，有些资料中省略了0 {{< /admonition >}}

## MySQL递归查询
MySQL 8.0 引入了递归公共表表达式（[Recursive Common Table Expressions](https://dev.mysql.com/doc/refman/8.0/en/with.html#common-table-expressions-recursive)），可以用来处理具有层级关系的数据，比如组织结构、文件系统目录等。
### 语法
与普通CTE的区别是递归CTE用`WITH RECURSIVE`引出，基本结构如下
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
### 用递归查询求阶乘和斐波那契数
求阶乘

这里定义临时表为Factorial，有两列：n和result，最终支取result的值
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

---

求斐波那契数

这是MySQL官方文档中的案例，n是序号，fib_n表示第n个斐波那契数，next_fib_n表示下一个斐波那契数
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

## 递归查询的应用

### 求下属层级

原题是[力扣3236](https://leetcode.cn/problems/ceo-subordinate-hierarchy/description/)题，这里只求下属层级，先不考虑工资问题

要求：编写一个解决方案来找到首席执行官的下属（**直接** 和 **非直接**），以及他们在 **等级制度中的级别**

```sql
Create table if not exists employees  
(  
    employee_id   int,  
    employee_name varchar(100),  
    manager_id    int  
);  
Truncate table Employees;  
insert into Employees (employee_id, employee_name, manager_id)  
values ('1', 'Alice', NULL);  
insert into Employees (employee_id, employee_name, manager_id)  
values ('2', 'Bob', '1');  
insert into Employees (employee_id, employee_name, manager_id)  
values ('3', 'Charlie', '1');  
insert into Employees (employee_id, employee_name, manager_id)  
values ('4', 'David', '2');  
insert into Employees (employee_id, employee_name, manager_id)  
values ('5', 'Eve', '2');  
insert into Employees (employee_id, employee_name, manager_id)  
values ('6', 'Frank', '3');  
insert into Employees (employee_id, employee_name, manager_id)  
values ('7', 'Grace', '3');  
insert into Employees (employee_id, employee_name, manager_id)  
values ('8', 'Helen', '5');
```

输入表：employees，Alice是首席执行官

- employee_id：员工id
- employee_name：员工姓名
- manager_id：上级id

| employee\_id | employee\_name | manager\_id |
| :----------- | :------------- | :---------- |
| 1            | Alice          | null        |
| 2            | Bob            | 1           |
| 3            | Charlie        | 1           |
| 4            | David          | 2           |
| 5            | Eve            | 2           |
| 6            | Frank          | 3           |
| 7            | Grace          | 3           |
| 8            | Helen          | 5           |

输出表：
- subordinate_id：下属id
- subprdinate_name：下属姓名
- hierarchy_level：下属相对于首席执行官的等级

| subordinate\_id | subordinate\_name | hierarchy\_level |
| :-------------- | :---------------- | :--------------- |
| 2               | Bob               | 1                |
| 3               | Charlie           | 1                |
| 4               | David             | 2                |
| 5               | Eve               | 2                |
| 6               | Frank             | 2                |
| 7               | Grace             | 2                |
| 8               | Helen             | 3                |

层级关系根据employee表中的employee_id和manager_id确定，这里的层级关系是这样的：
- Alice
	- Bob-1
		- David-2
		- Eve-2
			- Helen-3
	- Charile-1
		- Frank-2
		- Grace-2


实现

```sql
with recursive t as (select employee_id   subordinate_id,  
                            employee_name subordinate_name,  
                            0             hierarchy_level  
                     from employees  
                     where manager_id is null  
  
                     union all  
                     
                     select e.employee_id,  
                            e.employee_name,  
                            t.hierarchy_level + 1  
                     from employees e  
                              join t on e.manager_id = t.subordinate_id)  
select *  
from t  
where hierarchy_level != 0;
```

代码是这样运行的：

首先查询Alice作为初始化部分，把Alice的hierarchy_level定义为0
```sql
select employee_id   subordinate_id,  
       employee_name subordinate_name,  
       0             hierarchy_level  
from employees  
where manager_id is null
```

| subordinate\_id | subordinate\_name | hierarchy\_level |
| :-------------- | :---------------- | :--------------- |
| 1               | Alice             | 0                |

递归部分

把递归部分拆分成三步：关联、选择字段、union

1. 关联

关联employees表和临时表，关联条件是manager_id = subordinate_id，关联后的表是这样的：

| employee_id | employee_name | manager_id | subordinate_id | subordinate_name | hierarchy_level |
|-------------|---------------|------------|----------------|------------------|-----------------|
| 2           | Bob           | 1          | 1              | Alice            | 0               |
| 3           | Charlie       | 1          | 2              | Alice            | 1               |


2. 选择字段
- e.employee_id → 2和3
- e.employee_namet → Bob和Charlie
- hierarchy_level + 1 → 0 + 1

得到下表

| subordinate_id | subordinate_name | hierarchy_level |
| -------------- | ---------------- | --------------- |
| 2              | Bob              | 1               |
| 3              | Charlie          | 1               |

3. union

把初始化得到的表和上一步得到的表做union，得到下表

| subordinate_id | subordinate_name | hierarchy_level |
| -------------- | ---------------- | --------------- |
| 1              | Alice            | 0               |
| 2              | Bob              | 1               |
| 3              | Charlie          | 1               |

这个表作为临时表，继续执行递归操作，直到遍历完employees表中所有数据，最后用where条件排除hierarchy_level=0的员工就得到输出接过了
