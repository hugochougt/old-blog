---
layout: post
title: "为 octopress 配置404页面"
date: 2013-07-06 16:30
tags: [octopress]
---

老早就想为自己的 blog 配置一下404 not found page 了。上网搜索了一下，成果如下：[zhaqiang 404](http://zhaqiang.github.io/404/)。下面说明下配置过程。

在 `octopress/source` 目录下新建一个名为 "404.html" 的文件，并添加以下内容：

    ---
    layout:
    title: 404 not found
    footer: false
    comments: false
    categories:
    ---

然后就可以在文件后面添加你的个性化 404 not found 提示语了。受到陈浩老师[酷壳](http://coolshell.cn)的启发，我自己也弄了一个404公益页面。

参考链接：

- [Octopress配置404页面](http://yrzhll.com/blog/2012/12/18/404)
