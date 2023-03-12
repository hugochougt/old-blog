---
layout: post
title: "Vim 多窗口基本操作"
date: 2014-06-21 15:35
tags: [Vim, multiple windows]
---

从命令行打开多个文件时就为每个文件分配一个窗口，使用 `-o`（horizontally，横排） 或 `-O`（vertically，竖排）参数：

    $ vim -o file1 file2
    $ vim -O file1 file3

在编辑文件的时候打开新的窗口使用 `:split` 或 `:vsplit` 命令。

在窗口间移动：`CTRL-W` + 上下左右，`CTRL-W CTRL-W` 移动到下一个窗口。

移动窗口：`CTRL-W` + 大写的 HJKL。

Resize:

 *  `CTRL-W =` 將所有窗口变成同样的大小
 *  `CTRL-W -n` 將当前窗口减小 n 行。同理，`+` 增加 n 行（`:resize +n` 有同样的效果）
 *  `CTRL-W <` 和 `CTRL-W >` 分别减少和增加窗口的宽度

以上的 Vim 操作足够应付多数情况下的多窗口操作需求了。

本文内容参考自 [Learning the vi and Vim Editors](http://book.douban.com/subject/3041178/), page 174-186

**-EOF-**
