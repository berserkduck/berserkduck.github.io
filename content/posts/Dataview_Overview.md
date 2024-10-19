---
title: Dataview-概述
date: 2024-10-09T02:38:09+08:00
lastmod: 2024-10-17T15:09:23+08:00
author: Ryan
summary: Dataview文档翻译-概述
draft: false
categories:
  - Obsidian
  - Dataview
---


## 说明
[Dataivew](https://github.com/blacksmithgu/obsidian-dataview)是一个[Obsidian](https://obsidian.md/)插件，提供了一种结构化查询语言来对Obsidian库中内容进行查询。为了方便学习Dataview，对[Dataview文档](https://blacksmithgu.github.io/obsidian-dataview/)进行翻译。由于本人水平和精力有限，无法保证翻译完全准确。本文仅供参考，未来可能不会随官方文档同步更新，请务必查看官方文档。

---
[原文](https://blacksmithgu.github.io/obsidian-dataview/)

Dataview 是一个面向个人知识库的实时索引和查询引擎。用户可以在笔记中[**添加元数据**](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)，并使用[**Dataview 查询语言**](https://blacksmithgu.github.io/obsidian-dataview/queries/structure/)对笔记进行**查询**，来实现对内容的展示、过滤、排序和分组。Dataview 可以使查询始终保持最新状态，并且能轻松实现数据聚合。

Dataview可以实现包括但不限于以下操作：

- 跟踪日记中的睡眠情况，并自动创建每周睡眠时间表。
- 自动汇总笔记中书籍的链接，并按评分排序。
- 自动汇总与今天日期相关的页面，并显示在日记中。
- 查找并跟踪没有标签或带有特定标签的页面。
- 创建动态视图，显示即将到来的生日或笔记中记录的事件

{{< admonition type=note title="note" open=true >}} Dataview 提供了一种快速查询、显示和操作库中索引数据的方法！ {{< /admonition >}}  

Dataview 具有高度通用性和高性能，可以轻松扩展到数十万条带注释的笔记。

如果内置[查询语言](https://blacksmithgu.github.io/obsidian-dataview/query/queries/)不能满足需求，可以使用[Dataview API](https://blacksmithgu.github.io/obsidian-dataview/api/intro/)运行JavaScript ，可以在笔记中构建可能需要的任意程序。

{{< admonition type=info title="Dataview 用于显示，而不是编辑" open=true >}} Dataview 用于显示和计算数据，不会修改笔记和元数据（[任务](https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/#task-queries)除外） {{< /admonition >}}  

## 如何使用 Dataview

Dataview 由两大部分组成：**数据索引**和**数据查询**。

{{< admonition type=info title="更多细节请参阅文档中链接的页面" open=true >}} 以下小结将介绍Dataview可以实现的功能以及实现方法。请务必访问链接的页面以了解更多信息。  {{< /admonition >}}  


### 数据索引（Data Indexing）

Dataview 操作的是 Markdown 文件中的元数据。Dataview只能读取库中的特定数据。有些内容比如标签和要点（bullet points）（包括任务）可[自动](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/#implicit-fields)在 Dataview 中使用。使用**字段**添加其他数据，可以通过 [YAML Frontmatter](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/#frontmatter)将其添加到文件头部，也可以使用[Inline Fields](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/#inline-fields)将字段添加到笔记中，语法：`[key:: value]`。Dataview会*索引*这些数据以便查询。

{{< admonition type=note title="note" open=true >}} Dataview 会索引[某些信息](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/#implicit-fields)，比如标签，列表项以及通过字段添加的数据。只有索引过数据才可用于 Dataview 查询！  {{< /admonition >}}  

举例，某文件可能长这样：

```
--- 
author: "Edgar Allan Poe" 
published: 1845 
tags: poems 
--- 

# The Raven 

Once upon a midnight dreary, while I pondered, weak and weary, Over many a quaint and curious volume of forgotten lore—
```

或者这样：

```
#poems 

# The Raven 

From [author:: Edgar Allan Poe], written in (published:: 1845) 

Once upon a midnight dreary, while I pondered, weak and weary, Over many a quaint and curious volume of forgotten lore—
```

就索引元数据而言，以上两种方式没有区别，只是注释样式不同，可以按个人偏好[注释元数据](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)。在这个文件中，`author`可以作为**元数据字段** ，另外 Dataview 会自动将某些内容[作为隐式字段](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-pages/)，比如标签或标题。

{{< admonition type=note title="数据需要被索引" open=true >}} 在上面的例子中， Dataview并不索引诗歌本身：因为它是一个段落，不是元数据字段，也不是 Dataview 自动索引的内容。这段诗歌不是 Dataview 索引的一部分，因此无法查询。 {{< /admonition >}}  

### 数据查询（Data Querying）

你可以使用**查询**（Queries）获取**索引过的数据**。

编写查询语句的**三种方式**：[Dataview查询语言](https://blacksmithgu.github.io/obsidian-dataview/queries/dql-js-inline/#dataview-query-language-dql)、[行内语句](https://blacksmithgu.github.io/obsidian-dataview/queries/dql-js-inline#inline-dql)或[Javascript 查询](https://blacksmithgu.github.io/obsidian-dataview/queries/dql-js-inline#dataview-js)。

**Dataview 查询语言**( **DQL** ) 功能强大，可以用来查询、显示和操作数据。使用[**行内查询**](https://blacksmithgu.github.io/obsidian-dataview/queries/dql-js-inline#inline-dql)可以在笔记中的任意位置显示查询，行内查询也可以用于计算。有了**DQL**，你可能就不需要Javascript了。

DQL由这些部分组成：

- [**查询类型**](https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/)：指定输出样式
- [**FROM 语句**](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands#from)：零个或一个，用于选择的特定标签或文件夹（或其他[来源](https://blacksmithgu.github.io/obsidian-dataview/reference/sources/)）
- [**其他数据命令**](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands/)：零个或多个，用于过滤、分组和排序

例如，查询可以如下所示：

<pre><code>```dataview 
LIST 
```</code></pre>

这会列出库中所有的文件。

{{< admonition type=info title="除查询类型外，其他都是可选的" open=true >}} DQL 必须指定查询类型（[CALENDAR](https://blacksmithgu.github.io/obsidian-dataview/queries/query-types#calendar-queries)类型必须包含日期字段。）  {{< /admonition >}}  

可以添加更多过滤条件：

<pre><code>```dataview
LIST 
FROM #poems 
WHERE author = "Edgar Allan Poe"
```</code></pre>

这条查询会显示库中所有包含`#poems`标签，且`author` [字段](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)的值为`Edgar Allan Poe`的文件。这条查询可以找到上文中提到的示例页面。

`LIST`只是可选四种[查询类型](https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/)之一。例如，使用`TABLE`可以输出更多信息：

<pre><code>```dataview
TABLE author, published, file.inlinks AS "Mentions" 
FROM #poems 
```</code></pre>

输出：

| File (3)          | author           | published  | Mentions             |
|-------------------|------------------|------------|----------------------|
| The Bells         | Edgar Allan Poe  | 1849       |                      |
| The New Colossus  | Emma Lazarus     | 1883       | - [[Favorite Poems]] |
| The Raven         | Edgar Allan Poe  | 1845       | - [[Favorite Poems]] |

但这并非Dataview的全部功能。你还可以借助[**函数**](https://blacksmithgu.github.io/obsidian-dataview/reference/functions/)**对数据进行操作**。请注意，这些操作只是查询 - **文件内容不会被修改**。

<pre><code>```dataview 
TABLE author, date(now).year - published AS "Age in Yrs", length(file.inlinks) AS "Counts of Mentions" 
FROM #poems 
```</code></pre>

输出

| File (3)          | author           | Age in Yrs  | Count of Mentions |
|-------------------|------------------|-------------|-------------------|
| The Bells         | Edgar Allan Poe  | 173         | 0                 |
| The New Colossus  | Emma Lazarus     | 139         | 1                 |
| The Raven         | Edgar Allan Poe  | 177         | 1                 |

{{< admonition type=info title="info" open=true >}} [在此](https://blacksmithgu.github.io/obsidian-dataview/resources/examples/)查看更多示例。
 {{< /admonition >}}  

Dataview 不仅可以快速聚合数据并使结果保持最新状态，还提供了查看数据的全新视角。浏览文档以了解更多信息。

使用愉快！

## 资源和帮助

本文档并不是唯一可以帮助你完成数据之旅的地方。请查看[资源和支持](https://blacksmithgu.github.io/obsidian-dataview/resources/resources-and-support/)，获取有用的页面和视频。
