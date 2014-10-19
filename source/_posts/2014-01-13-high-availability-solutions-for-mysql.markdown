---
layout: post
title: "MySQL 高可用解决方案简介"
date: 2014-01-13 20:34
comments: true
categories: [MySQL, High Availability]
---

因工作需要近段时间收集并阅读了不少关于 MySQL HA（High Availability，高可用）的资料，现在整理一下做个总结，主要是为自己写的，但同时也希望会对搜索到本文的读者有帮助。

MySQL HA 的关键点不外乎**数据同步**和**失效切换**这两点。对于数据同步，MySQL 本身就有异步复制和半同步复制的功能（半同步复制在 MySQL 5.5之后的版本才有）。异步和半同步复制都会有数据不一致的风险，要保证数据一致性，就只能采用共享存储的方法。共享存储这一块我也没有做太多的调研，不过猜测使用最广泛的应该是 DRBD（Distributed Replicated Block Device），因为 NAS 和 SAN 等共享存储方案的成本太高。至于失效切换（failover），我了解到的就只有 [Keepalived](http://www.keepalived.org/) 和 [Heartbeat](http://www.linux-ha.org/wiki/Heartbeat)。一般来说 Keepalived 会搭配 MySQL Master-Master replication 的架构，而 Heartbeat 则搭配 DRBD。

另外两个较为独立的 HA 解决方案是 [Percona](www.percona.com) 公司开发的 Percona XtraDB Cluster 和 MySQL 本身的 MySQL Cluster。

在你选择一个高可用性方案前，你需要先问问自己这个问题：

**你是否真的需要 MySQL HA？**

如果你的回答是 Yes，那么接下来你还有一堆问题要回答：

 1. 你需要哪个级别的 HA？
 2. 你能不能忍受数据丢失？
 3. 你的应用是否使用了 MyISAM 存储引擎特有的特性？
 4. 应用的 write load 如何？
 5. 你规划的数据库容量多大？
 6. 你或你员工的水平有多高？
 7. 预算有多少？

以上7个问题翻译自 [Finding your MySQL High-Availability solution - The questions](http://www.mysqlperformanceblog.com/2009/10/16/finding-your-mysql-high-availability-solution-%E2%80%93-the-questions/)，该 blog 中有对各个问题的总结性解答，虽然写于2009，但我觉得现在还是有很高的参考价值。文后的评论也值得一读。

想了解各种主流 MySQL HA 方案的简述和优缺点，可阅读 [Choosing a MySQL High Availability Solution](http://www.percona.com/resources/technical-presentations/choosing-mysql-high-availability-solution-webinar-choosing-high) 这个 slide 和这篇 blog [High-availability options for MySQL, October 2013 update](http://www.mysqlperformanceblog.com/2013/10/23/high-availability-options-for-mysql-october-2013-update/)，然后再 google 其关键词搜索方案的详细原理和配置步骤等信息。

总的来说，MySQL HA 没有通用的解决方案，每个方案都有各自的优缺点，你需要根据自己应用的实际情况，选择合适的 HA 方案。

--------

自己前后也尝试搭建过几个 MySQL HA 方案，Percona XtraDB Cluster 没有搭建成功，DRBD + Heartbeat 的方案不适合在虚拟机上做测试，也就没有花时间弄了。搭建成功的就只有 MySQL Master-Master replication + Keepalived 的架构。后来了解到半同步复制，也搭建成功了。然后就使用 MySQL 自带的 sql-bench 工具分别对异步复制和半同步复制的性能进行了测试。测试完后，发现在内网网络环境较好的情况下，半同步复制并没有使 MySQL 的性能有明显的下降。所以，在应用提交的多数是小事务，并且主机之间的网络延迟较小的情况下，应优先选择配置半同步复制实现 MySQL HA，以大大减少出现数据不一致的风险。

本来想贴出性能比较的数据表的，不过 google 了一下，发现 Octopress 不支持表格，那就算了。

**-EOF-**

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")
