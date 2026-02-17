---
title: 导出WordPress文章为Markdown文件
published: 2026-02-15
category: 实用工具
tags: ["WordPress", "Astro", "Markdown"]
draft: false
---

## 概述

从WordPress转移到Astro时，需要把WordPress导出的XML转换为Markdown文件给Astro使用。

## 使用

### 前置准备

- 安装好`Node.js`
- 使用"All Content"导出的WordPress文件

### 安装及运行

> [!TIP]
> 如果在导出文件同一目录下运行直接指定名字就好

```zsh "Yes" "No" "XX" "XXXX" "......" "\/output"
npx wordpress-export-to-markdown

Need to install the following packages:
wordpress-export-to-markdown@3.0.4
Ok to proceed? (y) y

Starting wizard...
✓ Path to WordPress export file? cedon.WordPress.2026-02-15.xml
✓ Put each post into its own folder? Yes
✓ Add date prefix to posts? Yes
✓ Organize posts into date folders? No
✓ Save images? All Images

Parsing...
XX normal posts found.
XX pages found.
XX attached images found.
XX images scraped from post body content.

Saving posts...
XX posts to save.
✓ [post] XXXX
✓ [post] XXXX
✓ [post] XXXX
......
Done, got them all!

Saving images...
XX images to save.
✓ [image] XXXX
✓ [image] XXXX
✓ [image] XXXX
......
Done, got them all!

All done!
Look for your output files in: /output
```

## References
- [Github wordpress-export-to-markdown](https://github.com/lonekorean/wordpress-export-to-markdown)
