---
layout: post
title: "在 Linux 下使用二进制包安装 PostgreSQL"
date: 2013-11-13 20:43
comments: true
categories: [PostgreSQL] 
---

今天在公司的内网中终于成功使用编译好的二进制包安装 PostgreSQL，记录一下，以免下次安装又忘记了操作步骤。本文应该对于那些机器不能联网、但已经有了编译好的 PostgreSQL 安装包、却不知道如何配置和启动 PostgreSQL 的同学有帮助。

编译好的 PostgreSQL 安装包大概具有以下的目录结构：
    bin/
    lib/
    share/
    include/

本文假设 PostgreSQL 安装在 $PGSQL_HOME 目录中。

## 创建 data 目录
首先创建一个用于存放 PostgreSQL 数据库系统数据的文件夹，一般在 PostgreSQL 的根目录下创建就好了。

## 初始化数据库
使用 `initdb` 命令初始化数据。最好能指定一些参数，例如：

    initdb -D $PGSQL_HOME/data -A trust -U <username> -W

其中 -D 指示保存数据库数据的目录，-A 指示可以从本机连接数据库，-U 指示数据库的初始超级用户名， -W 指示需要设置密码。回车后就需要你输入两次密码了，根据提示来做就好。更详细的参数选项可用 `initdb --help` 查看。

## 启动数据库服务
运行 `pg_ctl start -D $PGSQL_HOME/data` 命令就可以，没什么可说的。完成后用 `ps -ef | grep postgre` 确认后台进程成功运行。

## 配置 LD_LIBRARY_PATH
编辑 $HOME/.bashrc 文件，添加 $PGSQL_HOME/lib 目录到 $LD_LIBRARY_PATH 环境变量中：

    export LD_LIBRARY_PATH=$PGSQL_HOME/lib:$LD_LIBRARY_PATH

然后 `source $HOME/.bashrc` 使环境变量生效。这个对于正确配置 PostgreSQL 是关键一步，之前我死活不能通过客户端连上 PostgreSQL 就是没有配置这个环境变量。没有配这个环境变量，会在执行创建数据库和连接数据库的时候出现 **undefined symbol PQconnectdbParams** 的错误。

## 创建数据库
使用 `createdb <dbname>` 命令来创建数据库。

## 访问数据库
使用 `psql <dbname>` 命令来访问数据库。

**-EOF-**

------

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")

如果觉得本文对你有帮助，可以请作者喝[咖啡](http://me.alipay.com/zhaqiang)。
