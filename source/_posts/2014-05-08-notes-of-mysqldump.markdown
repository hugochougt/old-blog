---
layout: post
title: "mysqldump 命令的一些用法"
date: 2014-05-08 19:23
comments: true
categories: [mysql, mysqldump]
---

导出某个数据库的所有表

    mysqldump -uroot -p database > db_backup.sql

这个默认会导出 triggers，但是不导出 stored routines 和 events。不想导出 triggers 请添加 `--skip-triggers` 参数。

完整备份（包括 stored routines 和 events）整个数据库

    mysqldump -uroot -p -R -E database > db_backup.sql

`-R` 表示导出 stored routines （包括了 functions 和 procedures），`-E` 表示导出 events。

只想导出表结构而不要数据，可以加 `-d` 参数

    mysqldump -uroot -p -d database > db_backup.sql

这是最近几天的工作需要修改表字段，经常需要备份测试数据库而用到的一些命令参数，在此记录一下。

另外，查看了 mysqldump 命令帮助信息（使用 `mysqldump --help` 列出），发现它的说明还是很详尽的，再记录一下自己认为重要的默认设置，及其开启/关闭选项。

`--opt` 默认开启，其相当添加了下列的参数：--add-drop-table, --add-locks, --create-options, --quick, --extended-insert, --lock-tables, --set-charset, --disable-keys。这个看英文也能大致知道其作用，就不一一解释了。要关闭该选项，使用 `--skip-opt`。

`--add-drop-table` 默认开启，就是在每一条 CREATE 命令前加上 DROP TABLE。

最后就是前面提到的 triggers、stored routines 和 events了，这些都容易在备份数据库的时候被忽略，还请注意。

**-EOF-**

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")
