```python
import time  # 时间库
import numpy as np  # numpy库
import pandas as pd  # pandas库
from pyecharts.charts import Bar3D # 3D柱形图
```

# 读取数据

原始数据文件中有五个sheet，把sheet名放到一个列表中


```python
sheet_names = ["2015", "2016", "2017", "2018", "会员等级"]

sheet_data = [pd.read_excel("../data/h_sales.xlsx", sheet_name=sheet_name) for sheet_name in sheet_names]
```

# 数据预处理
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



# 添加间隔日期列

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



# 计算RFM原始值

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



# 查看数据分布
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



# 自定义区间边界

分别把rfm划分为3个区间
> 注意：起始边界小于最小值


```python
r_bins = [-1, 79, 255, 365] 
f_bins = [0, 2, 5, 130] # f数据的分布比较极端，所以这里采用较小的值
m_bins = [0, 69, 1199, 206252]
```

# 对RFM分箱，计算得分


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



# 生成RFM组合
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



# 准备绘图


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


# 显示图形


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




<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
        <div id="218b5dc8bc4a49178e6fcbcedfef7f5f" class="chart-container" style="width:900px; height:500px;"></div>
    <script>
        var chart_218b5dc8bc4a49178e6fcbcedfef7f5f = echarts.init(
            document.getElementById('218b5dc8bc4a49178e6fcbcedfef7f5f'), 'white', {renderer: 'canvas'});
        var option_218b5dc8bc4a49178e6fcbcedfef7f5f = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "color": [
        "#c23531",
        "#2f4554",
        "#61a0a8",
        "#d48265",
        "#749f83",
        "#ca8622",
        "#bda29a",
        "#6e7074",
        "#546570",
        "#c4ccd3",
        "#f05b72",
        "#ef5b9c",
        "#f47920",
        "#905a3d",
        "#fab27b",
        "#2a5caa",
        "#444693",
        "#726930",
        "#b2d235",
        "#6d8346",
        "#ac6767",
        "#1d953f",
        "#6950a1",
        "#918597"
    ],
    "series": [
        {
            "type": "bar3D",
            "data": [
                [
                    111,
                    2015,
                    2180
                ],
                [
                    111,
                    2016,
                    1498
                ],
                [
                    111,
                    2017,
                    3169
                ],
                [
                    111,
                    2018,
                    2271
                ],
                [
                    112,
                    2015,
                    3811
                ],
                [
                    112,
                    2016,
                    2958
                ],
                [
                    112,
                    2017,
                    5737
                ],
                [
                    112,
                    2018,
                    4340
                ],
                [
                    113,
                    2015,
                    1785
                ],
                [
                    113,
                    2016,
                    1297
                ],
                [
                    113,
                    2017,
                    2506
                ],
                [
                    113,
                    2018,
                    1947
                ],
                [
                    121,
                    2017,
                    1
                ],
                [
                    121,
                    2018,
                    8
                ],
                [
                    122,
                    2015,
                    27
                ],
                [
                    122,
                    2017,
                    12
                ],
                [
                    122,
                    2018,
                    976
                ],
                [
                    123,
                    2015,
                    24
                ],
                [
                    123,
                    2017,
                    4
                ],
                [
                    123,
                    2018,
                    1904
                ],
                [
                    132,
                    2018,
                    2
                ],
                [
                    133,
                    2015,
                    15
                ],
                [
                    133,
                    2016,
                    233
                ],
                [
                    133,
                    2017,
                    36
                ],
                [
                    133,
                    2018,
                    188
                ],
                [
                    211,
                    2015,
                    3532
                ],
                [
                    211,
                    2016,
                    4678
                ],
                [
                    211,
                    2017,
                    3509
                ],
                [
                    211,
                    2018,
                    7304
                ],
                [
                    212,
                    2015,
                    6576
                ],
                [
                    212,
                    2016,
                    8983
                ],
                [
                    212,
                    2017,
                    6983
                ],
                [
                    212,
                    2018,
                    14297
                ],
                [
                    213,
                    2015,
                    2962
                ],
                [
                    213,
                    2016,
                    4036
                ],
                [
                    213,
                    2017,
                    2983
                ],
                [
                    213,
                    2018,
                    6389
                ],
                [
                    221,
                    2015,
                    2
                ],
                [
                    221,
                    2016,
                    6
                ],
                [
                    221,
                    2017,
                    3
                ],
                [
                    221,
                    2018,
                    13
                ],
                [
                    222,
                    2015,
                    43
                ],
                [
                    222,
                    2016,
                    69
                ],
                [
                    222,
                    2017,
                    46
                ],
                [
                    222,
                    2018,
                    216
                ],
                [
                    223,
                    2015,
                    37
                ],
                [
                    223,
                    2016,
                    42
                ],
                [
                    223,
                    2017,
                    38
                ],
                [
                    223,
                    2018,
                    253
                ],
                [
                    232,
                    2016,
                    4
                ],
                [
                    232,
                    2018,
                    9
                ],
                [
                    233,
                    2015,
                    20
                ],
                [
                    233,
                    2016,
                    50
                ],
                [
                    233,
                    2017,
                    589
                ],
                [
                    233,
                    2018,
                    387
                ],
                [
                    311,
                    2015,
                    1718
                ],
                [
                    311,
                    2016,
                    2171
                ],
                [
                    311,
                    2017,
                    2164
                ],
                [
                    311,
                    2018,
                    3221
                ],
                [
                    312,
                    2015,
                    3310
                ],
                [
                    312,
                    2016,
                    4438
                ],
                [
                    312,
                    2017,
                    4325
                ],
                [
                    312,
                    2018,
                    6582
                ],
                [
                    313,
                    2015,
                    1440
                ],
                [
                    313,
                    2016,
                    1947
                ],
                [
                    313,
                    2017,
                    1946
                ],
                [
                    313,
                    2018,
                    3008
                ],
                [
                    321,
                    2015,
                    6
                ],
                [
                    321,
                    2016,
                    9
                ],
                [
                    321,
                    2017,
                    2
                ],
                [
                    321,
                    2018,
                    17
                ],
                [
                    322,
                    2015,
                    61
                ],
                [
                    322,
                    2016,
                    86
                ],
                [
                    322,
                    2017,
                    64
                ],
                [
                    322,
                    2018,
                    199
                ],
                [
                    323,
                    2015,
                    44
                ],
                [
                    323,
                    2016,
                    64
                ],
                [
                    323,
                    2017,
                    60
                ],
                [
                    323,
                    2018,
                    198
                ],
                [
                    331,
                    2017,
                    1
                ],
                [
                    332,
                    2015,
                    3
                ],
                [
                    332,
                    2016,
                    8
                ],
                [
                    332,
                    2017,
                    2
                ],
                [
                    332,
                    2018,
                    24
                ],
                [
                    333,
                    2015,
                    15
                ],
                [
                    333,
                    2016,
                    28
                ],
                [
                    333,
                    2017,
                    87
                ],
                [
                    333,
                    2018,
                    355
                ]
            ],
            "label": {
                "show": false,
                "position": "top",
                "margin": 8
            }
        }
    ],
    "legend": [
        {
            "data": [
                ""
            ],
            "selected": {},
            "show": true,
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0
    },
    "visualMap": {
        "show": true,
        "type": "continuous",
        "min": 0,
        "max": 14297,
        "inRange": {
            "color": [
                "#313695",
                "#4575b4",
                "#74add1",
                "#abd9e9",
                "#e0f3f8",
                "#ffffbf",
                "#fee090",
                "#fdae61",
                "#f46d43",
                "#d73027",
                "#a50026"
            ]
        },
        "calculable": true,
        "splitNumber": 5,
        "orient": "vertical",
        "showLabel": true,
        "itemWidth": 20,
        "itemHeight": 140
    },
    "xAxis3D": {
        "name": "\u5206\u7ec4\u540d\u79f0",
        "nameGap": 20,
        "type": "category",
        "axisLabel": {
            "margin": 8
        }
    },
    "yAxis3D": {
        "name": "\u5e74\u4efd",
        "nameGap": 20,
        "type": "category",
        "axisLabel": {
            "margin": 8
        }
    },
    "zAxis3D": {
        "name": "\u4f1a\u5458\u6570\u91cf",
        "nameGap": 20,
        "type": "value",
        "axisLabel": {
            "margin": 8
        }
    },
    "grid3D": {
        "boxWidth": 200,
        "boxHeight": 100,
        "boxDepth": 80,
        "viewControl": {
            "autoRotate": false,
            "autoRotateSpeed": 10,
            "rotateSensitivity": 1
        }
    },
    "title": [
        {
            "text": "RFM\u5206\u7ec4\u7ed3\u679c"
        }
    ]
};
        chart_218b5dc8bc4a49178e6fcbcedfef7f5f.setOption(option_218b5dc8bc4a49178e6fcbcedfef7f5f);
    </script>
</body>
</html>




# 计算RFM分组会员数占比


```python
# 1 按 RFM_Group 分组之后，对number求和，返回新的df
rfm_persent = display_data.groupby(['rfm_group'])['number'].sum().reset_index()

# 2 计算当前RFM_Group分组中会员总数百分比，并添加列 count_per
rfm_persent['count_per'] = rfm_persent['number'] / display_data['number'].sum()

# 3 转换为百分数，保留2位小数
rfm_persent['count_per'] = rfm_persent['count_per'].apply(lambda x : format(x, '.2%'))

# 4 按number列进行排序，由大到小
rfm_persent.sort_values('number', ascending=False, inplace=True)

rfm_persent
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
      <th>rfm_group</th>
      <th>number</th>
      <th>count_per</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>212</td>
      <td>36839</td>
      <td>24.79%</td>
    </tr>
    <tr>
      <th>8</th>
      <td>211</td>
      <td>19023</td>
      <td>12.80%</td>
    </tr>
    <tr>
      <th>17</th>
      <td>312</td>
      <td>18655</td>
      <td>12.55%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>112</td>
      <td>16846</td>
      <td>11.34%</td>
    </tr>
    <tr>
      <th>10</th>
      <td>213</td>
      <td>16370</td>
      <td>11.02%</td>
    </tr>
    <tr>
      <th>16</th>
      <td>311</td>
      <td>9274</td>
      <td>6.24%</td>
    </tr>
    <tr>
      <th>0</th>
      <td>111</td>
      <td>9118</td>
      <td>6.14%</td>
    </tr>
    <tr>
      <th>18</th>
      <td>313</td>
      <td>8341</td>
      <td>5.61%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>113</td>
      <td>7535</td>
      <td>5.07%</td>
    </tr>
    <tr>
      <th>5</th>
      <td>123</td>
      <td>1932</td>
      <td>1.30%</td>
    </tr>
    <tr>
      <th>15</th>
      <td>233</td>
      <td>1046</td>
      <td>0.70%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>122</td>
      <td>1015</td>
      <td>0.68%</td>
    </tr>
    <tr>
      <th>24</th>
      <td>333</td>
      <td>485</td>
      <td>0.33%</td>
    </tr>
    <tr>
      <th>7</th>
      <td>133</td>
      <td>472</td>
      <td>0.32%</td>
    </tr>
    <tr>
      <th>20</th>
      <td>322</td>
      <td>410</td>
      <td>0.28%</td>
    </tr>
    <tr>
      <th>12</th>
      <td>222</td>
      <td>374</td>
      <td>0.25%</td>
    </tr>
    <tr>
      <th>13</th>
      <td>223</td>
      <td>370</td>
      <td>0.25%</td>
    </tr>
    <tr>
      <th>21</th>
      <td>323</td>
      <td>366</td>
      <td>0.25%</td>
    </tr>
    <tr>
      <th>23</th>
      <td>332</td>
      <td>37</td>
      <td>0.02%</td>
    </tr>
    <tr>
      <th>19</th>
      <td>321</td>
      <td>34</td>
      <td>0.02%</td>
    </tr>
    <tr>
      <th>11</th>
      <td>221</td>
      <td>24</td>
      <td>0.02%</td>
    </tr>
    <tr>
      <th>14</th>
      <td>232</td>
      <td>13</td>
      <td>0.01%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>121</td>
      <td>9</td>
      <td>0.01%</td>
    </tr>
    <tr>
      <th>6</th>
      <td>132</td>
      <td>2</td>
      <td>0.00%</td>
    </tr>
    <tr>
      <th>22</th>
      <td>331</td>
      <td>1</td>
      <td>0.00%</td>
    </tr>
  </tbody>
</table>
</div>



# 导出数据


```python
rfm_gb.to_excel('sales_rfm_score1.xlsx')  # 保存数据为Excel
```


```python
# 需要安装pymysql，部分版本需要额外安装sqlalchemy
# 导入sqlalchemy的数据库引擎
from sqlalchemy import create_engine

# 创建数据库引擎，传入url规则的字符串
engine = create_engine('mysql+pymysql://root:123456@192.168.88.100:3306/test?charset=utf8')
# mysql+pymysql://root:123456@192.168.88.100:3306/test?charset=utf8
# mysql 表示数据库类型
# pymysql 表示python操作数据库的包
# root:123456 表示数据库的账号和密码，用冒号连接
# 192.168.88.100:3306/test 表示数据库的ip和端口，以及名叫test的数据库
# charset=utf8 规定编码格式

# df.to_sql()方法将df数据快速写入数据库
rfm_gb.to_sql('rfm_gb', engine, index=False, if_exists='append')
# 第一个参数为数据表的名称
# 第二个参数engine为数据库交互引擎
# index=False 表示不添加自增主键
```

<iframe src = 'https://berserkduck.github.io/RFM模型.html'
width = '900px'
height = '500px'
frameborder = '1'
allowfullscreen = 'true'
</iframe>
