---
title: "同期群分析"
date: 2024-09-26T22:23:13+08:00
lastmod: 2024-09-26T22:23:13+08:00
author: "Ryan"
summary: "同期群分析及案例"
draft: false            
categories: ["Pandas"]
---
---
## 同期群

 Cohort在 [剑桥在线词典](https://dictionary.cambridge.org/zhs/%E8%AF%8D%E5%85%B8/%E8%8B%B1%E8%AF%AD-%E6%B1%89%E8%AF%AD-%E7%AE%80%E4%BD%93/cohort#:~:text=COHORT%E7%BF%BB%E8%AF%91) 中的中文释义是“有共同特点（通常指年龄）的一群人”，这里译为**同期群**。比如2000年出生的人可以划分为一个同期群，1月份新增的用户也可以划分为一个同期群
## 同期群分析

同期群分析（Cohort Analysis）指将用户按初始行为的发生时间分组，分析群组行为随时间变化的趋势

举一个例子

下表记录了某电商平台2024年1月-4月每月新增用户数，以及该月新增用户在之后月份留存率

在这个表格中，把每一行作为一个同期群，研究新增用户在一段时间后的留存率

| 月份      | 当月新增用户 | +1月 | +2月 | +3月 | +4月 |
| ------- | ------ | --- | --- | --- | --- |
| 2024年1月 | 112    | 46% | 38% | 32% | 5%  |
| 2024年2月 | 235    | 53% | 41% | 39% |     |
| 2024年3月 | 367    | 39% | 35% |     |     |
| 2024年4月 | 400    | 47% |     |     |     |
| 2024年5月 | 367    |     |     |     |     |
- 2024年1月新增112个用户
	- 第二个月有46%的回头客
		- 第三个月有38%的回头客
			- 第四个月有5%的回头客
其他月同理

{{< admonition type=note title="Note" open=true >}}  
在每一行中，留存率是对当月新增用户而言的

留存率=某月复购用户数/对应期新增用户数

{{< /admonition >}}  



通过跟踪和比较不同群体的行为模式，我们可以更好地理解用户行为随时间的变化趋势，从而制定更有效的策略

同期群分析的维度和指标可以根据实际情况调整，比如以天为维度统计用户留存情况或统计回购客单价等

## 案例一

>计算单月同期群留存情况


Excel表中存有2023年1-12月的订单数据，假设1月份的用户都是新用户，求1月和2月的留存客户数
### 导入数据

```python
import pandas as pd

df = pd.read_excel('./Cohort_Analysis.xlsx')
df.head()
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>主订单编号</th>
      <th>用户ID</th>
      <th>付款时间</th>
      <th>实付金额</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>73465136654</td>
      <td>uid135460366</td>
      <td>2023-01-01 09:32:12</td>
      <td>166</td>
    </tr>
    <tr>
      <th>1</th>
      <td>73465136655</td>
      <td>uid135460367</td>
      <td>2023-01-01 09:11:50</td>
      <td>117</td>
    </tr>
    <tr>
      <th>2</th>
      <td>73465136656</td>
      <td>uid135460368</td>
      <td>2023-01-01 11:49:02</td>
      <td>166</td>
    </tr>
    <tr>
      <th>3</th>
      <td>73465136657</td>
      <td>uid135460369</td>
      <td>2023-01-01 12:20:24</td>
      <td>77</td>
    </tr>
    <tr>
      <th>4</th>
      <td>73465136658</td>
      <td>uid135460370</td>
      <td>2023-01-01 01:23:15</td>
      <td>158</td>
    </tr>
  </tbody>
</table>
</div>

### 计算1月和2月的订单量


```python
# 增加year_month列
df['year_month'] = df['付款时间'].astype(str).str[:7] 

Jan = '2023-01'
Feb = '2023-02'
Jan_orders = df[df['year_month'] == Jan]
Feb_orders = df[df['year_month'] == Feb]
print('一月份的订单量', len(Jan_orders))
print('二月份的订单量', len(Feb_orders))

```

```
一月份的订单量 12039
二月份的订单量 4114
```


### 计算1月和2月的下单用户数
```python
Jan_customers = Jan_orders.groupby('用户ID')['实付金额'].sum().reset_index()
Feb_customers = Feb_orders.groupby('用户ID')['实付金额'].sum().reset_index()
print('一月的用户数量', len(Jan_customers))
print('二月的用户数量', len(Feb_customers))
```

```
一月的用户数量 8193
二月的用户数量 3313
```

### 计算月新增用户数


```python
Jan_new_customers = Jan_customers['用户ID']
print('2023年1月新增用户：', len(Jan_new_customers))
```

```
2023年1月新增用户： 8193
```


### 计算2月新增用户数


```python
Feb_new_customers = Feb_customers.loc[Feb_customers['用户ID'].isin(Jan_orders['用户ID']) == False, :]
print('2023年2月新增用户：', len(Feb_new_customers))
```

```
2023年2月新增用户： 2740
```


### 计算1月留存情况

```python
Jan_retention = []
for i in ['2023-02', '2023-03', '2023-04', '2023-05', '2023-06', '2023-07', '2023-08', '2023-09', '2023-10', '2023-11',
          '2023-12']:
    next_month_orders = df.loc[df['year_month'] == i, :]
    target_users_Jan = Jan_customers[Jan_customers['用户ID'].isin(next_month_orders['用户ID'])].groupby('用户ID')[
        '实付金额'].sum().reset_index()
    Jan_retention.append([i + '留存情况：', len(target_users_Jan)])

Jan_retention
```

```
[['2023-02留存情况：', 573],
 ['2023-03留存情况：', 1601],
 ['2023-04留存情况：', 1050],
 ['2023-05留存情况：', 1079],
 ['2023-06留存情况：', 1906],
 ['2023-07留存情况：', 815],
 ['2023-08留存情况：', 1102],
 ['2023-09留存情况：', 863],
 ['2023-10留存情况：', 628],
 ['2023-11留存情况：', 1049],
 ['2023-12留存情况：', 372]]
```

### 计算2月留存情况


```python
Feb_retention = []
for i in ['2023-03', '2023-04', '2023-05', '2023-06', '2023-07', '2023-08', '2023-09', '2023-10', '2023-11', '2023-12']:
    next_month_orders = df.loc[df['year_month'] == i, :]
    target_users_Feb = Feb_new_customers.loc[Feb_new_customers['用户ID'].isin(next_month_orders['用户ID']), :]
    Feb_retention.append([i + '留存情况：', len(target_users_Feb)])

Feb_retention
```

```
[['2023-03留存情况：', 558],
 ['2023-04留存情况：', 340],
 ['2023-05留存情况：', 379],
 ['2023-06留存情况：', 587],
 ['2023-07留存情况：', 293],
 ['2023-08留存情况：', 317],
 ['2023-09留存情况：', 267],
 ['2023-10留存情况：', 205],
 ['2023-11留存情况：', 304],
 ['2023-12留存情况：', 112]]
```

### 加入新增数据 得到完整1月份同期群结果


```python
Jan_retention.insert(0, ['2023年1月新增用户：', len(Jan_new_customers)])
Jan_retention
```

```
[['2023年1月新增用户：', 8193],
 ['2023-02留存情况：', 573],
 ['2023-03留存情况：', 1601],
 ['2023-04留存情况：', 1050],
 ['2023-05留存情况：', 1079],
 ['2023-06留存情况：', 1906],
 ['2023-07留存情况：', 815],
 ['2023-08留存情况：', 1102],
 ['2023-09留存情况：', 863],
 ['2023-10留存情况：', 628],
 ['2023-11留存情况：', 1049],
 ['2023-12留存情况：', 372]]
```



### 加入新增数据 得到完整2月份同期群结果


```python
Feb_retention.insert(0, ['2023年2月新增用户：', len(Feb_new_customers)])
Feb_retention
```

```
[['2023年2月新增用户：', 2740],
 ['2023-03留存情况：', 558],
 ['2023-04留存情况：', 340],
 ['2023-05留存情况：', 379],
 ['2023-06留存情况：', 587],
 ['2023-07留存情况：', 293],
 ['2023-08留存情况：', 317],
 ['2023-09留存情况：', 267],
 ['2023-10留存情况：', 205],
 ['2023-11留存情况：', 304],
 ['2023-12留存情况：', 112]]
```

