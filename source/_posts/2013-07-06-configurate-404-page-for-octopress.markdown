---
layout: post
title: "为 octopress 配置404页面"
date: 2013-07-06 16:30
comments: true
categories: [octopress config]
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
[Octopress配置404页面](http://yrzhll.com/blog/2012/12/18/404)

**-EOF-**

-----

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")

如果觉得本文对你有帮助，可以猛击下面图片↓↓↓↓↓↓↓↓无压力小额赞助作者喝咖啡。

<center><a href='http://me.alipay.com/zhaqiang'><img src='https://img.alipay.com/sys/personalprod/style/mc/btn-index.png' style="border:none;vertical-align:middle;"/></a></center>


