---
title: 笔记随想
date: 2024-09-22T15:29:29+08:00
lastmod: 2024-09-22T15:29:35+08:00
author: Ryan
summary: 
draft: false
categories:
  - ""
  - Obsidian
---
---
## 从笔记说起

我使用[Onsidian](https://obsidian.md/)已经一年多，目前有三个库：
- Archive：以前的笔记，目前暂时不用
- Notes：正在用的库
- content：连接了本站的content文件夹，用来编写发布内容

其中大大小小的笔记有几百篇，主要以学习笔记为主，最近考虑对笔记做一些发布。
## 发布
发布方式有很多，目前我了解的有：
### 官方的发布服务
我觉得官方发布服务最大的优势支持目录和反向链接，可以保留笔记在Obsidian中的原始结构，比如[官方的帮助文档](https://publish.obsidian.md/help-zh/%E7%94%B1%E6%AD%A4%E5%BC%80%E5%A7%8B)，但是价格太贵，放弃。
### Netlify + Obsidian Digital Garden
用[Netlify](https://www.netlify.com/)配合[Obsidian Digital Garden插件](https://github.com/oleeskild/Obsidian-Digital-Garden)搭建发布网站，实现起来非常简单，具体可参考这篇帖子：[Obsidian 最简单的发布分享方式](https://forum-zh.obsidian.md/t/topic/19256)。用这种方式有很多优势，比如可以保留原始目录，可以在插件中对网站做很多自定义，网站搭好后用ODG这个插件一键发布，相当便捷。但是这个方案有个致命的问题：Nerlify提供的免费服务器虽然能用，但文章多了后网站会非常卡顿，归根结底还是钱没到位。由于我暂时不考虑购买自己的服务器，所以放弃这个方案。

另外推荐一个在应用该部署方案时发现的一个很漂亮的[网站](https://notes.johnmavrick.com)，一开始也是用ODG实现的，不过目前该网站已转为使用官方发布服务。
### Hugo + Github Pages
搭建一个静态网页配合Github Pages实现发布，访问速度可以，可自定义程度高，最终我选择这个方案，Thus Ryan’s Blog was born!🐣

## 使用
经过一番折腾，Markdown功能基本都实现了，剩下的就是写笔记了。

由于发布内容主要来自Notes库，所以我试着把Notes库中的内容全部放到了content目录，但是Hugo会把content目录里的所以文档都编译发布，所以只好又把两个库分开。

另一个要折腾地方的是Hugo的[Front matter](https://gohugo.io/content-management/front-matter/)，也就是Obsidian中的文档属性，用于保存一些文档的基本信息，以前不太关注文档的属性，但是发布需要

一种方法是在Hugo的archetypes目录中的default.md文件中定义好常用属性，用`hugo new content`命令生成新文章，但是每次写东西都要先跑到命令行实在不便。由于Obsidian支持模板，所以我写了一个模板文件，其中属性如下：
```md
---
title: "<% tp.file.title %>"
date: <% tp.file.creation_date("YYYY-MM-DDTHH:mm:ss+08:00") %>
lastmod: <% tp.date.now("YYYY-MM-DDTHH:mm:ss+08:00") %>
author: "Ryan"
summary: ""
draft: false            
categories: [""]
---
---
## 
```
更多参数可以参考[Templater官方文档](https://silentvoid13.github.io/Templater/)

使用方法是在Obsidian中新建一个笔记，修改笔记名称，然后点击Templater按钮选择这个模板，会自动添加上面的内容到新建的笔记中

还剩下一个小问题：lastmod的时间问题，没找到好的解决方法，暂时先手动修改吧

