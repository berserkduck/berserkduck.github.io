---
title: Dataview-在笔记中添加元数据
date: 2024-10-22T15:22:30+08:00
lastmod: 2024-10-22T15:28:59+08:00
author: Ryan
summary: Dataview文档翻译-在笔记中添加元数据
draft: false
categories:
  - Obsidian
  - Dataview
---
---

[原文](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/)

Dataview不能查询库中所有内容。为了进行搜索、过滤和显示，需要为笔记内容添加**索引**。某些内容会自动被索引，比如bullet points或任务列表——即所谓的**隐式字段** （**Implicit fields**），下文会详细介绍——而其他数据则需要指定元数据**字段**。

## “字段”什么是？

元数据字段（metadata field）是一对**键**和**值**。字段的*值*具有数据类型（更多信息请见[此处](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/)），数据类型决定了字段查询方式。

可以在**笔记**、**列表**或**任务**中添加任意数量的字段。

## 如何添加字段？

可以通过三种不同的方式向**笔记**添加字段。字段的显示方式取决于添加方式。

在**任务或列表**中可以访问YAML前置信息（YAML Frontmatter），但不能把前置信息添加到特定列表中。如果需要在列表或任务中添加元数据，请使用[行内字段](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/#inline-fields)。

### 前置信息

前置信息（Frontmatter）是Markdown中常用的语法扩展，可以把YAML格式的元数据添加到页面顶部。 Obsidian 原生支持前置信息，在其[官方文档](https://help.obsidian.md/Advanced+topics/YAML+front+matter)中有详细介绍。Dataview会把所有YAML前置信息识别成字段。

```
    ---
    alias: "document"
    last-reviewed: 2021-08-17
    thoughts:
      rating: 8
      reviewable: false
    ---
```

如果笔记中定义了上面的前置信息，那么笔记中就包含了`alias`、`last-reviewed`和`thoughts`字段。每个字段都有不同的**数据类型**：

- `alias`是[文本](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#text)类型，因为被""包裹
- `last-reviewed`是[日期](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#date)类型，因为符合ISO日期格式
- `thoughts`是一个[对象](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#object)类型，因为使用YAML前置信息对象语法

可以使用以下查询语句来查询这个笔记：

````
```dataview
LIST
WHERE thoughts.rating = 8
```
````

### 行内字段

如果想要更自然地注释，Dataview提供了一种“行内”字段，语法为`Key:: Value`，可以在笔记中任何地方使用。这样就可以在需要查询的时候定义字段——比如在句子中间。

如果行内字段独占一行，即前后没有内容，可以这样写：

```
# Markdown Page

Basic Field:: Some random Value
**Bold Field**:: Nice!
```

`::`之后的所有内容都是字段的值，直到换行。

{{< admonition type=note title="注意`::`" open=true >}} 请注意，使用行内字段时，需要在键和值之间使用双冒号`::`，而 YAML 前置信息中的字段用一个冒号即可。 {{< /admonition >}} 

如果需要在句子中定义元数据或在同一行中添加多个字段，可以把字段括在方括号里：

```
I would rate this a [rating:: 9]! It was [mood:: acceptable].
```

{{< admonition type=info title="列表和任务中的字段" open=true >}} 
使用元数据注释列表项目（比如任务）时，需要使用括号语法（因为该字段不是此行中的唯一信息）

```
- [ ] Send an mail to David about the deadline [due:: 2022-04-05].
```

括号内的行内字段是明确向特定列表项添加字段的唯一方法，YAML 前置内容始终适用于整个页面（但也可用于列表项的上下文中。）
{{< /admonition >}} 

还有另一种括号语法，在阅读模式下会隐藏键：

```
This will not show the (longKeyIDontNeedWhenReading:: key).
```

这句话会被显示成：

`This will not show the key.`

可以在一个笔记中同时使用YAML前置信息和行内字段。无需决定使用哪一种，可以视情况混合使用。

## 字段名称

如果一个笔记中使用了上文提到的所有行内字段，那么可以使用的元数据如下：

| Metadata Key                  | Sanitized Metadata key        | Value                      | Data Type of Value |
| ----------------------------- | ----------------------------- | -------------------------- | ------------------ |
| `Basic Field`                 | `basic-field`                 | Some random Value          | Text               |
| `Bold Field`                  | `bold-field`                  | Nice!                      | Text               |
| `rating`                      | -                             | 9                          | Number             |
| `mood`                        | -                             | acceptable                 | Text               |
| `due`                         | -                             | Date Object for 2022-04-05 | Date               |
| `longKeyIDontNeedWhenReading` | `longkeyidontneedwhenreading` | key                        | Text               |


如表所示，如果在键中包含**空格或大写字母**，Dataview会将该键进行**转换**。

**带空格的键**不能用于查询。有两种选择：要么使用经过转换的键名，即全部小写并用破折号代替空格；要么使用**行变量**。具体请参考[FAQ](https://blacksmithgu.github.io/obsidian-dataview/resources/faq/)。译注：行变量 `row["Field With Space In It"]`

如果愿意也可以按原样使用**带有大写字母的键**。经过转换键在查询时不区分大小写，用起来更方便：例如，使用转换过键`somemetadata`可以同时查询不同文件中的`someMetaData`和`someMetadata`字段。

还有一点，**粗体字段的键的格式没有了**。尽管`**`是键名的一部分，但在作为索引时会被忽略。所有其他内置格式也是如此，例如删除线或斜体。格式化的键只能按原始格式查询。利用这一机制可以在笔记中创建带格式的键。

### 表情符号和非拉丁字符的使用

字段名不只限于拉丁字符。可以使用 UTF-8 中可用的所有字符：

```
Noël:: Un jeu de console
クリスマス:: 家庭用ゲーム機
[🎅:: a console game]
[xmas🎄:: a console game]
```

**可以用表情符号作为元数据的键**，但有一些限制。如果字段名包含表情符号，需要把表情符号写在方括号里才能被Dataview识别。另外需要注意，在切换操作系统（比如从 Windows 切换到 Android）时，同一个表情符号可能会使用另一个字符编码，在查询时可能会找不到元数据。

{{< admonition type=info title="任务字段简写" open=true >}}任务中的[简写语法](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/#field-shorthands)是例外。可以使用简写而无需括号。但请注意，这仅适用于列出的简写 - 其他所有字段（无论是否带有表情符号）都需要使用该`[key:: value]`语法。
  {{< /admonition >}} 
## 隐式字段

即使没有在笔记中显式地添加任何元数据，Dataview 也会提供大量现成的索引数据。下面是一些隐式字段（implicit fields）的示例：

- 文件创建日期（`file.cday`）
- 文件中的链接（`file.outlinks`）
- 文件中的标签 ( `file.etags`)
- 文件中的所有列表（`file.lists`和`file.tasks`）

可用的隐式字段因您查看的是页面还是列表项而异。有关可用隐式字段的完整列表，请参阅[页面元数据](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-pages/)和[任务和列表元数据](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/)。
