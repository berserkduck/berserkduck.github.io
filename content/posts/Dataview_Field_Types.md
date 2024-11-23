---
title: Dataview-字段类型
date: 2024-10-25T14:24:35+08:00
lastmod: 2024-10-25T14:25:35+08:00
author: Ryan
summary: Dataview文档翻译-字段类型
draft: false
categories:
  - Dataview
---
---

[原文](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/)
# 字段类型

Dataview的所有字段都有**类型**，字段的类型决定了字段在Dataview中的显示和操作方式。请参阅[“添加元数据”](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)来创建字段，并确定哪些字段会自动显示在[页面元数据](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-pages/)和[任务和列表元数据](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/)中。

## 字段类型很重要

Dataview除了可以编写各种复杂查询以外，还提供了一些可用于修改元数据的[函数](https://blacksmithgu.github.io/obsidian-dataview/reference/functions/)。某些函数需要传入特定的数据类型。也就是说，字段的类型决定了字段支持的函数。另外，类型不同，查询结果的展示方式也会有所不同。

用户一般不需要关注字段的类型，但如果想要做计算或者其他骚操作，就不得不关注了。

{{< admonition type=info title="根据类型进行不同的渲染" open=true >}} 

如果你有这样的文件：
````
date1:: 2021-02-26T15:15
date2:: 2021-04-17 18:00

```dataview
TABLE date1, date2
WHERE file = this.file
```
````

可以得到以下查询结果（取决于Dataview的日期和时间格式设置）：

| File (1)   | date1                       | date2            |
|------------|-----------------------------|------------------|
| Untitled 2 | 3:15 PM - February 26, 2021 | 2021-04-17 18:00 |

`date1`被识别为**日期**，而`date2`是普通**文本**，`date1`被解析成日期类型。 在下方了解有关[日期](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#date)的更多信息。

{{< /admonition >}}


## Dataview的字段类型

Dataview有以下几种常用的字段类型。

### 文本类型

字段默认都是文本类型。如果字段没有指定具体的类型，则为文本。

```
Example:: This is some normal text.
```


{{< admonition type=note title="多行文本" open=true >}}
多行文本作为值只能通过YAML前置信息和管道运算符实现：

```
--- 
poem: |   
	Because I could not stop for Death,
	He kindly stopped for me;
	The carriage held but just ourselves
	And Immortality.
author: "[[Emily Dickinson]]"
title: "Because I could not stop for Death" 
---
```

对于行内字段，换行符意味着值的结束。

{{< /admonition >}}

### 数字类型

例如"6"和"3.6"

```
Example:: 6
Example:: 2.4
Example:: -80
```

在YAML中，数字类型不加引号：

```
---
rating: 8
description: "A nice little horror movie"
---
```

### 布尔类型

在编程中布尔类型只有两个值：true或false

```
Example:: true
Example:: false
```

### 日期类型

符合[ISO8601](https://en.wikipedia.org/wiki/ISO_8601)表示法的文本会自动转换为日期对象。ISO8601格式如下`YYYY-MM[-DDTHH:mm:ss.nnn+ZZ]`。月份后面的部分可选。

```
Example:: 2021-04
Example:: 2021-04-18
Example:: 2021-04-18T04:19:35.000
Example:: 2021-04-18T04:19:35.000+06:30
```

查询以上日期时，可以访问日期的某一部分属性：

- field.year
- field.month
- field.weekyear
- field.week
- field.weekday
- field.day
- field.hour
- field.minute
- field.second
- field.millisecond

举例，如果要访问日期中的月份，可以用`datefield.month`：

````
birthday:: 2001-06-11 

```dataview 
LIST birthday 
WHERE birthday.month = date(now).month
```
````


以上查询会返回所有在本月的生日。更多`date(now)`的细节见[Literals](https://blacksmithgu.github.io/obsidian-dataview/reference/literals/#dates)。

{{< admonition type=info title="显示日期对象" open=true >}}

Dataview会以人性化格式显示日期对象，即`3:15 PM - February 26, 2021`。可以在 Dataview 的“View”设置下使用“Date Forma”和“Date + Time Format”调整此格式的外观。如果你只想修改查询中的格式，可以用[dateformat函数](https://blacksmithgu.github.io/obsidian-dataview/reference/functions/#dateformatdatedatetime-string)。
{{< /admonition >}}

### 持续时间类型

持续时间类型的格式为`<time> <unit>`，例如`6 hours`或`4 minutes`。可以用常见的英文缩写 ，例如`6hrs`或`2m`。可以在一个字段中指定多个时间单位，例如`6hr 4min`，也可以使用逗号：`6 hours, 4 minutes`

```
Example:: 7 hours
Example:: 16days
Example:: 4min
Example:: 6hr7min
Example:: 9 years, 8 months, 4 days, 16 hours, 2 minutes Example:: 9 yrs 8 min
```
查找被识别为[literals](https://blacksmithgu.github.io/obsidian-dataview/reference/literals/#durations)持续时间的值的完整列表。

{{< admonition type=note title="日期和持续时间的运算" open=true >}}


日期类型和持续时间类型相互兼容。可以将持续时间添加到日期来生成新日期：

```
departure:: 2022-10-07T15:15 
length of travel:: 1 day, 3 hours 
**Arrival**: = `this.departure + this.length-of-travel`
```

并且当你使用日期计算时你会得到一个持续时间：

```
release-date:: 2023-02-14T12:00 
`= this.release-date - date(now)` until release!!
```

好奇吗？阅读[Literals](https://blacksmithgu.github.io/obsidian-dataview/reference/literals/#dates)`date(now)`下的更多信息。

{{< /admonition >}}

### 链接类型

Obsidian的链接格式：`[[Page]]`或`[[Page|Page Display]]`。

```
Example:: [[A Page]] 
Example:: [[Some Other Page|Render Text]]
```
{{< admonition type=info title="YAML中的链接" open=true >}}

如果在前置内容中引用链接，则需要加引号，如下所示：`key: "[[Link]]"`。这是Obsidian默认支持的格式。未加引号的链接在YAML中无效，无法进行解析。
```
---
parent: "[[parentPage]]"
---
```

要注意，这是一个Dataview的链接，而不是Obsidian的链接，也就是说该链接不会出现在出链中，不会显示在关系图谱中，也不会在重命名时更新。

{{< /admonition >}}

### 列表类型

列表是多值字段。在YAML中，多值字段被定义为YAML数组：

```
---
key3: [one, two, three]
key4:  
 - four  
 - five  
 - six
---
```

在行内字段中，多个值用逗号分隔：

```
Example1:: 1, 2, 3
Example2:: "yes", "or", "no"
```

需要注意，在行内字段中，将文本括括在引号里才能被识别成一个列表（如`Example2`）。`yes, or, no`被识别为纯文本。

{{< admonition type=info title="同一文件中的重复元数据键会导致列表" open=true >}}

如果你在一个笔记中使用同一个键两次或两次以上，Dataview会把该键对应的所有值合成一个列表。例如：
````
grocery:: flour
[...]
grocery:: soap

```dataview
LIST grocery WHERE file = this.file
``` 
````
会返回一个包含`flour`和`soap`的列表。

{{< /admonition >}}

{{< admonition type=note title="数组是列表" open=true >}}

在本文档的某些地方，你会看到术语“数组（Array）”。数组在Javascript中表示列表 - 列表和数组是一样的。需要数组作为参数的函数需要列表作为参数。

{{< /admonition >}}

### 对象类型

对象是一个父字段下有多个字典类型的子字段。对象只能在YAML中定义：

```
---
obj:   
  key1: "Val"   
  key2: 3   
  key3:      
    - "List1"     
    - "List2"     
    - "List3" 
---
```

在查询时，可以用`obj.key1`的格式访问字段值：

````
```dataview
TABLE obj.key1, obj.key2, obj.key3
WHERE file = this.file 
```
````
