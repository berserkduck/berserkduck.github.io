---
title: RFM模型
date: 2024-09-26T00:19:57+08:00
lastmod: 2024-09-26T00:30:07+08:00
author: Ryan
summary: ""
draft: false
categories:
  - ""
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
