+++
title = 'Jupyter导出文件测试'
+++
# 文件写入MySQL

```python
import pandas as pd

df = pd.read_csv('../output/student.csv', parse_dates=['birthday'])
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
      <th>id</th>
      <th>name</th>
      <th>birthday</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>张三</td>
      <td>1990-10-02</td>
      <td>34</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>李四</td>
      <td>2000-03-03</td>
      <td>24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>王五</td>
      <td>2005-12-23</td>
      <td>19</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>赵六</td>
      <td>1982-11-12</td>
      <td>42</td>
    </tr>
  </tbody>
</table>
</div>

```python
from sqlalchemy import create_engine

conn = create_engine('mysql+pymysql://root:123456@192.168.88.100:3306/test?charset=utf8')
```


```python
from sqlalchemy import types

birthday_dtype = {'birthday': types.Date}
df.to_sql('student', con=conn, if_exists='replace', index=False, dtype=birthday_dtype)
```

# 从MySQL读取

## 从表

```python
df = pd.read_sql('student', con=conn)
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
      <th>id</th>
      <th>name</th>
      <th>birthday</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>张三</td>
      <td>1990-10-02</td>
      <td>34</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>李四</td>
      <td>2000-03-03</td>
      <td>24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>王五</td>
      <td>2005-12-23</td>
      <td>19</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>隔壁老王</td>
      <td>1982-11-12</td>
      <td>42</td>
    </tr>
  </tbody>
</table>
</div>

## 从sql

```python
df = pd.read_sql_query('select * from student where age > 30', con=conn)
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
      <th>id</th>
      <th>name</th>
      <th>birthday</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>张三</td>
      <td>1990-10-02</td>
      <td>34</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>隔壁老王</td>
      <td>1982-11-12</td>
      <td>42</td>
    </tr>
  </tbody>
</table>
</div>
