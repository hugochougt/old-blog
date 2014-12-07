---
layout: post
title: "解决 Ubuntu 系统不显示顶部菜单栏和侧边栏问题"
date: 2013-10-25 20:54
comments: true
categories: [ubuntu]
---

更新了 Ubuntu 系统后，重启后登录先是出现了不能直接登录到图形桌面的问题。在控制台输入用户名和密码后，使用 `startx` 命令启动图形桌面。但是桌面又不显示顶部菜单栏和侧边栏。还好可以用 `Ctrl + Alt + T` 命令来启动终端。打开 chrome 后 google 了一下，发现了这篇 [blog](http://nerd-is.in/2013-08/solve-ubuntu-do-not-show-menubar-sidebar/) 和 blog 上提到的这个[帖子](http://forum.ubuntu.org.cn/viewtopic.php?f=94&t=333122)。最终用 `unity --reset` 这个命令解决了问题。

Ubuntu 已经不是第一次出现这个问题了。以前都是重启一两次就解决，这次忍了两天，终于找到了一个靠谱的解决方案。

 **-EOF-**
