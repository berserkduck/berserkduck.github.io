---
title: Quartz自定义折腾
date: 2025-05-05
---
以下是一些对Quartz的自定义修改记录

## 修改页面布局
Quartz的布局基本和Obsidian一样，左侧是目录树，中间是文档正文，右侧是关系图谱和文章目录。这个布局在`quartz.layout.ts`中定义

## 使用本地字体
修改`quartz.layout.ts`中的`fontOrigin`的值为`"local"`
```typescript
theme: {
      fontOrigin: "local",
      cdnCaching: true,
      typography: {
        header: "CustomHeaderFont",
        body: "CustomBodyFont",
        code: "CustomCodeFont",
      },
```
字体存放在`quartz/static/fonts`中，修改`quartz/styles/custom.css`，依次定义header、body和code的字体
```css
@font-face {
  font-family: "CustomHeaderFont";
  src: url("/static/fonts/LXGWWenKai-Regular.ttf") format("truetype");
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: "CustomBodyFont";
  src: url("/static/fonts/LXGWWenKai-Regular.ttf") format("truetype");
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: "CustomCodeFont";
  src: url("/static/fonts/JetBrainsMono-Regular.ttf") format("truetype");
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}
```