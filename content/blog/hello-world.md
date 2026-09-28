---
title: "第一篇文章：博客开张"
date: 2026-09-27T00:45:00+08:00
# URL 末段（不填则用标题生成，中文会变成一串百分号编码）
slug: "hello-world"
description: "博客搭起来了。这篇文章说明站点的结构，也是一份可以照抄的写作模板。"
tags: [
    "随笔",
]
categories: [
    "other",
]
---

博客搭起来了 🎉

<!--more-->

## 这个站点用什么搭的

- **Hugo**（extended 版）+ **hugo-bearneo** 主题，构建速度极快，页面很轻
- 内容用 Markdown 写，放在 `content/blog/` 目录
- 主题固件是 submodule，升级用 `git submodule update --remote`
- 推送到 GitHub 后由 GitHub Actions 自动构建并发布到 GitHub Pages

## 怎么写新文章

在站点根目录执行：

```bash
hugo new content blog/文章文件名.md
```

或者直接复制本文件的表头（front matter），改 `title` / `date` / `tags` 即可。

Front matter 里常用字段：

| 字段 | 作用 |
| --- | --- |
| `title` | 文章标题 |
| `date` | 发布时间（决定排序；写未来时间会当天不显示） |
| `slug` | 文章 URL 末段，例如 `/p/hello-world/` |
| `description` | 摘要，用于列表页和搜索引擎 |
| `tags` | 标签数组 |
| `draft: true` | 标记为草稿，默认不发布 |
| `mermaid: true` | 允许本文使用 Mermaid 图表 |

`<!--more-->` 之前的内容会作为列表页的摘要。

## 已开启的功能

按年份分组的文章列表、标题搜索、文章目录、图片点击放大、外链新窗口打开。

---

这篇文章可以删掉，或者留着当模板用。
