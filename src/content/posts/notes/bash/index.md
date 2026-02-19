---
title: Bash学习笔记以及一些常用快捷键
published: 2026-02-19
category: 学习笔记
tags: ["Bash"]
draft: true
---

## 常用快捷键

`Ctrl + L`：清除屏幕并将当前行移到页面顶部。
`Ctrl + C`：中止当前正在执行的命令。
`Shift + PageUp`：向上滚动。
`Shift + PageDown`：向下滚动。
`Ctrl + U`：从光标位置删除到行首。
`Ctrl + K`：从光标位置删除到行尾。
`Ctrl + W`：删除光标位置前一个单词。
`Ctrl + D`：关闭 Shell 会话。
`↑，↓`：浏览已执行命令的历史记录。

## 简介

bash 会忽略冗余的空格

```bash
echo this   is a      content

# output
this is a content
```

分号`;`能够分隔各个命令

```bash
echo 这是第一句;echo 这是第二句

# output
这是第一句
这是第二句
```

`&&`和`||`

`&&`连接前后命令时，需要前一个命令执行*成功*才会执行下一个命令

`||`连接前后命令时，需要前一个命令执行*失败*才会执行下一个命令

## 

## echo 命令
`-n`：取消换行
`-e`：解析引号内的特殊字符(如\\n)

## type 命令
