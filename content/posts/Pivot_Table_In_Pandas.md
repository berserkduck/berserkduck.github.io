---
title: Pandas数据透视表
date: 2024-09-22T10:25:40+08:00
lastmod:
  - <% tp.file.last_modified_date("YYYY-MM-DDTHH:mm:ss+08:00") %>
author: Ryan
summary: 
draft: false
categories:
  - Pandas
---
---
数据透视表是基于原数据表，按照一定规则呈现汇总数据，方便我们从不同维度去观察数据，Pandas中用`pivot_table`函数实现数据透视表

## 基本语法
```Python
df.pivot_table(
    values=None, 
    index=None, 
    columns=None, 
    aggfunc='mean',
    margins=False
)
```
- values：列名或列名列表，是要聚合的数据列，默认聚合所有数值列
- index：行索引
- columns：列索引
- aggfunc：可以传入内置函数或函数列表，默认为mean
- margins：默认为False，用于添加总计行和总计列

## 示例
```python
import pandas as pd

data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'],
    'Gender': ['Female', 'Male', 'Male', 'Male', 'Female', 'Male'],
    'Age': [25, 30, 35, 40, 45, 50],
    'Score': [85, 90, 80, 95, 88, 92]
}

df = pd.DataFrame(data)
df
```

|  | Name | Gender | Age | Score |
| :--- | :--- | :--- | :--- | :--- |
| 0 | Alice | Female | 25 | 85 |
| 1 | Bob | Male | 30 | 90 |
| 2 | Charlie | Male | 35 | 80 |
| 3 | David | Male | 40 | 95 |
| 4 | Eve | Female | 45 | 88 |
| 5 | Frank | Male | 50 | 92 |

按性别分组，求平均分
```python
result = df.pivot_table(values='Score', index='Gender', aggfunc='mean')
result
```

| Gender | Score |
| :----- | :---- |
| Female | 86.50 |
| Male   | 89.25 |

按性别和年龄分组求平均分
```python
result = df.pivot_table(values='Score', index=['Gender', 'Age'], aggfunc='mean')
result
```

| Gender | Age | Score |
| :----- | :-- | :---- |
| Female | 25  | 85    |
|        | 45  | 88    |
| Male   | 30  | 90    |
|        | 35  | 80    |
|        | 40  | 95    |
|        | 50  | 92    |

添加总计行和总计列
```python

result = df.pivot_table(values='Score', index='Gender', columns='Age', aggfunc='mean', margins=True)
result
```

| Age<br/>Gender | 25<br/> | 30<br/> | 35<br/> | 40<br/> | 45<br/> | 50<br/> | All<br/>  |
| :------------- | :------ | :------ | :------ | :------ | :------ | :------ | :-------- |
| Female         | 85.0    | NaN     | NaN     | NaN     | 88.0    | NaN     | 86.500000 |
| Male           | NaN     | 90.0    | 80.0    | 95.0    | NaN     | 92.0    | 89.250000 |
| All            | 85.0    | 90.0    | 80.0    | 95.0    | 88.0    | 92.0    | 88.333333 |

使用多个聚合函数
```python
result = df.pivot_table(values='Score', index='Gender', aggfunc=['mean', 'sum'])
result
```

| <br/><br/>Gender | mean<br/>Score<br/> | sum<br/>Score<br/> |
| :--------------- | :------------------ | :----------------- |
| Female           | 86.50               | 173                |
| Male             | 89.25               | 357                |
