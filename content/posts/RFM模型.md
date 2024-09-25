---
title: RFM模型
date: 2024-09-26T00:19:57+08:00
lastmod: 2024-09-26T02:00:07+08:00
author: Ryan
summary: ""
draft: false
categories:
  - "Pandas"
---
---
## RFM模型
RFM模型是衡量**客户价值**和**客户创造利益能力**的重要手段，RFM的含义如下：
- R：最近一次消费 (Recency)
- F：消费频率 (Frequency)
- M：消费金额 (Monetary)
通过对R、F、M三个维度的分析评估客户的订单活跃价值，实现客户分群或价值区分。

RFM模型基于一个固定时间点来做模型分析，不同时间计算的的RFM结果可能不一样。

一般可以把用户分为如下类别：

| R | F | M | 用户类别   |
|---|---|---|--------|
| 高 | 高 | 高 | 重要价值用户 |
| 高 | 低 | 高 | 重要发展用户 |
| 低 | 高 | 高 | 重要保持用户 |
| 低 | 低 | 高 | 重要挽留用户 |
| 高 | 高 | 低 | 一般价值用户 |
| 高 | 低 | 低 | 一般发展用户 |
| 低 | 高 | 低 | 一般保持用户 |
| 低 | 低 | 低 | 一般挽留用户 |
## RFM模型的基本实现过程
通过以下案例模拟RFM模型的实现过程

### 原始数据
```python
import pandas as pd

data = {
    '用户ID': [1, 2, 3, 4],
    '最近交易日期': ['2024-09-03', '2024-08-25', '2024-08-30', '2024-07-06'],
    '消费次数': [3, 5, 1, 12],
    '消费金额': [230, 310, 500, 100]
}
df = pd.DataFrame(data)
df['最近交易日期'] = df['最近交易日期'].astype('datetime64[ns]')
df
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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
    </tr>
  </tbody>
</table>
</div>



### 增加日期列
```python
from datetime import datetime

df['当前日期'] = datetime.now().strftime("%Y-%m-%d")
df['当前日期'] = df['当前日期'].astype('datetime64[ns]')
df
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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
      <th>当前日期</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
      <td>2024-09-26</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
      <td>2024-09-26</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
      <td>2024-09-26</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
      <td>2024-09-26</td>
    </tr>
  </tbody>
</table>
</div>


### 求时间间隔

```python
df['间隔天数'] = df['当前日期'] - df['最近交易日期']
df['间隔天数'] = df['间隔天数'].dt.days
df
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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
      <th>当前日期</th>
      <th>间隔天数</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
      <td>2024-09-26</td>
      <td>23</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
      <td>2024-09-26</td>
      <td>32</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
      <td>2024-09-26</td>
      <td>27</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
      <td>2024-09-26</td>
      <td>82</td>
    </tr>
  </tbody>
</table>
</div>


### 计算RFM

```python
df['R'] = df.apply(lambda row: 1 if row['间隔天数'] > 60 else 
                                    (2 if row['间隔天数'] > 30 else 
                                     (3 if row['间隔天数'] > 14 else 
                                      (4 if row['间隔天数'] > 7 else 5))), axis=1)


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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
      <th>当前日期</th>
      <th>间隔天数</th>
      <th>R</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
      <td>2024-09-26</td>
      <td>23</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
      <td>2024-09-26</td>
      <td>32</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
      <td>2024-09-26</td>
      <td>27</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
      <td>2024-09-26</td>
      <td>82</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['F'] = df.apply(lambda row: 5 if row['消费次数'] > 10 else 
                                    (4 if row['消费次数'] > 3 else 
                                     (3 if row['消费次数'] > 2 else 
                                      (2 if row['消费次数'] > 1 else 1))), axis=1)
df
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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
      <th>当前日期</th>
      <th>间隔天数</th>
      <th>R</th>
      <th>F</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
      <td>2024-09-26</td>
      <td>23</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
      <td>2024-09-26</td>
      <td>32</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
      <td>2024-09-26</td>
      <td>27</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
      <td>2024-09-26</td>
      <td>82</td>
      <td>1</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['M'] = df.apply(lambda row: 5 if row['消费金额'] > 1000 else 
                                    (4 if row['消费金额'] > 500 else 
                                     (3 if row['消费金额'] > 300 else 
                                      (2 if row['消费金额'] > 150 else 1))), axis=1)
df
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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
      <th>当前日期</th>
      <th>间隔天数</th>
      <th>R</th>
      <th>F</th>
      <th>M</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
      <td>2024-09-26</td>
      <td>23</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
      <td>2024-09-26</td>
      <td>32</td>
      <td>2</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
      <td>2024-09-26</td>
      <td>27</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
      <td>2024-09-26</td>
      <td>82</td>
      <td>1</td>
      <td>5</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>


### 生成RFM模型

```python
df['R'] = df.apply(lambda row: '高' if row['R'] > 2 else '低', axis=1)
df['F'] = df.apply(lambda row: '高' if row['F'] > 3 else '低', axis=1)
df['M'] = df.apply(lambda row: '高' if row['M'] > 2 else '低', axis=1)
df
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
      <th>用户ID</th>
      <th>最近交易日期</th>
      <th>消费次数</th>
      <th>消费金额</th>
      <th>当前日期</th>
      <th>间隔天数</th>
      <th>R</th>
      <th>F</th>
      <th>M</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2024-09-03</td>
      <td>3</td>
      <td>230</td>
      <td>2024-09-26</td>
      <td>23</td>
      <td>高</td>
      <td>低</td>
      <td>低</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2024-08-25</td>
      <td>5</td>
      <td>310</td>
      <td>2024-09-26</td>
      <td>32</td>
      <td>低</td>
      <td>高</td>
      <td>高</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2024-08-30</td>
      <td>1</td>
      <td>500</td>
      <td>2024-09-26</td>
      <td>27</td>
      <td>高</td>
      <td>低</td>
      <td>高</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2024-07-06</td>
      <td>12</td>
      <td>100</td>
      <td>2024-09-26</td>
      <td>82</td>
      <td>低</td>
      <td>高</td>
      <td>低</td>
    </tr>
  </tbody>
</table>
</div>

## RFM模型应用案例




```python
import time  # 时间库
import numpy as np  # numpy库
import pandas as pd  # pandas库
from pyecharts.charts import Bar3D # 3D柱形图
```

### 读取数据

原始数据文件中有五个sheet，把sheet名放到一个列表中


```python
sheet_names = ["2015", "2016", "2017", "2018", "会员等级"]

sheet_data = [pd.read_excel("../data/h_sales.xlsx", sheet_name=sheet_name) for sheet_name in sheet_names]
```

### 数据预处理
1. 把前四个df纵向合并为一个df
2. 删除包含缺失值的行数据
3. 保留订单金额大于1的行数据
4. 提取每年数据的最大提交日期值保存到新列year中
5. 对year分组，提取每组中提交日期列的最大值，保存到新列max_year_date中
> 这样方便后续针对每年的数据分别做RFM计算，而不是针对4年的数据统一做RFM计算


```python
# 汇总所有数据, 使用pandas.concat连接前四个dataframe
data_merge = pd.concat(sheet_data[:-1], axis=0)

# 删除包含缺失值的行数据
data_merge.dropna(inplace=True)

# 保留订单金额大于1的行数据
data_merge.query('订单金额 > 1', inplace=True)

# 提取 提交日期 列的 年份 保存到year新列中
data_merge['year'] = data_merge['提交日期'].dt.year

# 对year新列分组, 提取每组中提交日期列的最大值, 保存到max_year_date新列中, 
data_merge['max_year_date'] = data_merge.groupby(['year'])['提交日期'].transform('max')

data_merge
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
      <th>会员ID</th>
      <th>订单号</th>
      <th>提交日期</th>
      <th>订单金额</th>
      <th>year</th>
      <th>max_year_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15278002468</td>
      <td>3000304681</td>
      <td>2015-01-01</td>
      <td>499.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
    </tr>
    <tr>
      <th>1</th>
      <td>39236378972</td>
      <td>3000305791</td>
      <td>2015-01-01</td>
      <td>2588.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
    </tr>
    <tr>
      <th>2</th>
      <td>38722039578</td>
      <td>3000641787</td>
      <td>2015-01-01</td>
      <td>498.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11049640063</td>
      <td>3000798913</td>
      <td>2015-01-01</td>
      <td>1572.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
    </tr>
    <tr>
      <th>4</th>
      <td>35038752292</td>
      <td>3000821546</td>
      <td>2015-01-01</td>
      <td>10.1</td>
      <td>2015</td>
      <td>2015-12-31</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>81344</th>
      <td>39229485704</td>
      <td>4354225182</td>
      <td>2018-12-31</td>
      <td>249.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
    </tr>
    <tr>
      <th>81345</th>
      <td>39229021075</td>
      <td>4354225188</td>
      <td>2018-12-31</td>
      <td>89.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
    </tr>
    <tr>
      <th>81346</th>
      <td>39288976750</td>
      <td>4354230034</td>
      <td>2018-12-31</td>
      <td>48.5</td>
      <td>2018</td>
      <td>2018-12-31</td>
    </tr>
    <tr>
      <th>81347</th>
      <td>26772630</td>
      <td>4354230163</td>
      <td>2018-12-31</td>
      <td>3196.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
    </tr>
    <tr>
      <th>81348</th>
      <td>39455580335</td>
      <td>4354235084</td>
      <td>2018-12-31</td>
      <td>2999.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
    </tr>
  </tbody>
</table>
<p>202827 rows × 6 columns</p>
</div>



### 添加间隔日期列

用max_year_date - 提交日期


```python
# 计算日期间隔天数（该订单日期距离当年最后1天的天数），并添加列 date_interval（该列的数据类型为timedelta64[ns]）
data_merge['date_interval'] = data_merge['max_year_date'] - data_merge['提交日期']

# 转换日期间隔为数字 
data_merge['date_interval'] = data_merge['date_interval'].dt.days 

data_merge
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
      <th>会员ID</th>
      <th>订单号</th>
      <th>提交日期</th>
      <th>订单金额</th>
      <th>year</th>
      <th>max_year_date</th>
      <th>date_interval</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15278002468</td>
      <td>3000304681</td>
      <td>2015-01-01</td>
      <td>499.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
      <td>364</td>
    </tr>
    <tr>
      <th>1</th>
      <td>39236378972</td>
      <td>3000305791</td>
      <td>2015-01-01</td>
      <td>2588.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
      <td>364</td>
    </tr>
    <tr>
      <th>2</th>
      <td>38722039578</td>
      <td>3000641787</td>
      <td>2015-01-01</td>
      <td>498.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
      <td>364</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11049640063</td>
      <td>3000798913</td>
      <td>2015-01-01</td>
      <td>1572.0</td>
      <td>2015</td>
      <td>2015-12-31</td>
      <td>364</td>
    </tr>
    <tr>
      <th>4</th>
      <td>35038752292</td>
      <td>3000821546</td>
      <td>2015-01-01</td>
      <td>10.1</td>
      <td>2015</td>
      <td>2015-12-31</td>
      <td>364</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>81344</th>
      <td>39229485704</td>
      <td>4354225182</td>
      <td>2018-12-31</td>
      <td>249.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>81345</th>
      <td>39229021075</td>
      <td>4354225188</td>
      <td>2018-12-31</td>
      <td>89.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>81346</th>
      <td>39288976750</td>
      <td>4354230034</td>
      <td>2018-12-31</td>
      <td>48.5</td>
      <td>2018</td>
      <td>2018-12-31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>81347</th>
      <td>26772630</td>
      <td>4354230163</td>
      <td>2018-12-31</td>
      <td>3196.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>81348</th>
      <td>39455580335</td>
      <td>4354235084</td>
      <td>2018-12-31</td>
      <td>2999.0</td>
      <td>2018</td>
      <td>2018-12-31</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>202827 rows × 7 columns</p>
</div>



### 计算RFM原始值

按year、会员ID列分组，分别对date_interval、提交日期、订单金额做不同的运算
- date_interval：求最小值
- 提交日期：count
- 订单金额：sum

然后改列名
- date_interval --> R
- 提交日期 --> F
- 订单金额 --> M


```python
# as_index=False表示重置索引
rfm_gb = data_merge.groupby(['year', '会员ID'], as_index=False).agg({
    # 1.1 R 求分组后 date_interval 列中最小值：计算当年该会员最后一次订单距离年末12月31日的间隔天数
    "date_interval" : "min",
    # 1.2 F 订单频率，计算当年该会员一共消费多少次，也可以对 订单号 列进行count计算
    "订单号" : "count",
    # 1.3 M 计算订单总金额：计算当年该会员一共消费多少钱
    "订单金额" : "sum"
})

# 2 重命名列名 'year', '会员ID', 'r', 'f', 'm'
rfm_gb.columns = ['year', '会员ID', 'r', 'f', 'm']
rfm_gb
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
      <th>year</th>
      <th>会员ID</th>
      <th>r</th>
      <th>f</th>
      <th>m</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015</td>
      <td>267</td>
      <td>197</td>
      <td>2</td>
      <td>105.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2015</td>
      <td>282</td>
      <td>251</td>
      <td>1</td>
      <td>29.7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015</td>
      <td>283</td>
      <td>340</td>
      <td>1</td>
      <td>5398.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015</td>
      <td>343</td>
      <td>300</td>
      <td>1</td>
      <td>118.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2015</td>
      <td>525</td>
      <td>37</td>
      <td>3</td>
      <td>213.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>148586</th>
      <td>2018</td>
      <td>39538034299</td>
      <td>272</td>
      <td>1</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>148587</th>
      <td>2018</td>
      <td>39538034662</td>
      <td>189</td>
      <td>1</td>
      <td>3558.0</td>
    </tr>
    <tr>
      <th>148588</th>
      <td>2018</td>
      <td>39538035729</td>
      <td>179</td>
      <td>1</td>
      <td>3699.0</td>
    </tr>
    <tr>
      <th>148589</th>
      <td>2018</td>
      <td>39545237824</td>
      <td>275</td>
      <td>1</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>148590</th>
      <td>2018</td>
      <td>39546136285</td>
      <td>163</td>
      <td>1</td>
      <td>19.9</td>
    </tr>
  </tbody>
</table>
<p>148591 rows × 5 columns</p>
</div>



### 查看数据分布
1. 获取 r, f, m 列 的所有数据
2. 查看统计信息
3. 转置 T


```python
desc_pd = rfm_gb.iloc[:, 2:].describe().T
desc_pd
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
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>r</th>
      <td>148591.0</td>
      <td>165.524043</td>
      <td>101.988472</td>
      <td>0.0</td>
      <td>79.0</td>
      <td>156.0</td>
      <td>255.0</td>
      <td>365.0</td>
    </tr>
    <tr>
      <th>f</th>
      <td>148591.0</td>
      <td>1.365002</td>
      <td>2.626953</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>130.0</td>
    </tr>
    <tr>
      <th>m</th>
      <td>148591.0</td>
      <td>1323.741329</td>
      <td>3753.906883</td>
      <td>1.5</td>
      <td>69.0</td>
      <td>189.0</td>
      <td>1199.0</td>
      <td>206251.8</td>
    </tr>
  </tbody>
</table>
</div>



### 自定义区间边界

分别把rfm划分为3个区间
> 注意：起始边界小于最小值


```python
r_bins = [-1, 79, 255, 365] 
f_bins = [0, 2, 5, 130] # f数据的分布比较极端，所以这里采用较小的值
m_bins = [0, 69, 1199, 206252]
```

### 分箱计算得分


```python
# 对rfm_gb['r']的值按照r_bins进行划分，划分结果对应为新的值，新的值为labels中的对应值
# range函数：range(start, stop[, step])
# 1 计算 R 得分: r_score
rfm_gb['r_score'] = pd.cut(rfm_gb['r'], bins=r_bins, labels=[j for j in range(len(r_bins)-1, 0, -1)])

# 2 计算 F 得分: f_score
rfm_gb['f_score'] = pd.cut(rfm_gb['f'], bins=f_bins, labels=[i for i in range(1, len(f_bins))])

# 3 计算 M 得分: m_score
rfm_gb['m_score'] = pd.cut(rfm_gb['m'], bins=m_bins, labels=[i for i in range(1, len(m_bins))])

rfm_gb.head()
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
      <th>year</th>
      <th>会员ID</th>
      <th>r</th>
      <th>f</th>
      <th>m</th>
      <th>r_score</th>
      <th>f_score</th>
      <th>m_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015</td>
      <td>267</td>
      <td>197</td>
      <td>2</td>
      <td>105.0</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2015</td>
      <td>282</td>
      <td>251</td>
      <td>1</td>
      <td>29.7</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015</td>
      <td>283</td>
      <td>340</td>
      <td>1</td>
      <td>5398.0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015</td>
      <td>343</td>
      <td>300</td>
      <td>1</td>
      <td>118.0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2015</td>
      <td>525</td>
      <td>37</td>
      <td>3</td>
      <td>213.0</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



### 生成RFM组合
把r、f、m拼接成RFM


```python
# 1 r_score、f_score、m_score 三列转为str类型
rfm_gb['r_score'] = rfm_gb['r_score'].astype(np.str)
rfm_gb['f_score'] = rfm_gb['f_score'].astype(np.str)
rfm_gb['m_score'] = rfm_gb['m_score'].astype(np.str)

# 2 r_score、f_score、m_score 三列拼接字符串，并添加新列 rfm_group
rfm_gb['rfm_group'] = rfm_gb['r_score'] + rfm_gb['f_score'] + rfm_gb['m_score']

rfm_gb.head()
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
      <th>year</th>
      <th>会员ID</th>
      <th>r</th>
      <th>f</th>
      <th>m</th>
      <th>r_score</th>
      <th>f_score</th>
      <th>m_score</th>
      <th>rfm_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015</td>
      <td>267</td>
      <td>197</td>
      <td>2</td>
      <td>105.0</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>212</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2015</td>
      <td>282</td>
      <td>251</td>
      <td>1</td>
      <td>29.7</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>211</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015</td>
      <td>283</td>
      <td>340</td>
      <td>1</td>
      <td>5398.0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>113</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015</td>
      <td>343</td>
      <td>300</td>
      <td>1</td>
      <td>118.0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>112</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2015</td>
      <td>525</td>
      <td>37</td>
      <td>3</td>
      <td>213.0</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>322</td>
    </tr>
  </tbody>
</table>
</div>



### 准备绘图


```python
# 1 根据 rfm_group 和 year 分组 再求 每组会员人数
display_data = rfm_gb.groupby(['rfm_group', 'year'])['会员ID'].count().reset_index()

# 2 重命名 'rfm_group','year','number'
display_data.columns = ['rfm_group', 'year', 'number']

# 3 将 rfm_group 转成 np.int32 类型
display_data['rfm_group'] = display_data['rfm_group'].astype(np.int32)

display_data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 88 entries, 0 to 87
    Data columns (total 3 columns):
     #   Column     Non-Null Count  Dtype
    ---  ------     --------------  -----
     0   rfm_group  88 non-null     int32
     1   year       88 non-null     int64
     2   number     88 non-null     int64
    dtypes: int32(1), int64(2)
    memory usage: 1.8 KB


### 显示图形


```python
from pyecharts.globals import CurrentConfig, NotebookType
CurrentConfig.NOTEBOOK_TYPE = NotebookType.JUPYTER_LAB
CurrentConfig.ONLINE_HOST

from pyecharts.commons.utils import JsCode
import pyecharts.options as opts

# 颜色池
range_color = ['#313695', '#4575b4', '#74add1', '#abd9e9', '#e0f3f8', '#ffffbf',
               '#fee090', '#fdae61', '#f46d43', '#d73027', '#a50026']

range_max = int(display_data['number'].max())

c = (
    Bar3D()#设置了一个3D柱形图对象
    .add(
        "",#图例
        [d.tolist() for d in display_data.values],#数据
        xaxis3d_opts=opts.Axis3DOpts(type_="category", name='分组名称'),#x轴数据类型，名称，rfm_group
        yaxis3d_opts=opts.Axis3DOpts(type_="category", name='年份'),#y轴数据类型，名称，year
        zaxis3d_opts=opts.Axis3DOpts(type_="value", name='会员数量'),#z轴数据类型，名称，number
    )
    .set_global_opts( # 全局设置
        visualmap_opts=opts.VisualMapOpts(max_=range_max, range_color=range_color), #设置颜色，及不同取值对应的颜色
        title_opts=opts.TitleOpts(title="RFM分组结果"),#设置标题
    )
)
# c.render("RFM模型.html") #输出图标
c.render_notebook()
```

<iframe src = 'https://berserkduck.github.io/RFM模型.html'
width = '900px'
height = '500px'
frameborder = '1'
allowfullscreen = 'true'
</iframe>

