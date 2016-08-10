---
layout: post
title: "基于腾讯云服务的 Rails 负载均衡部署方案简介"
date: 2016-07-29 20:32
comments: true
---

最近应客户要求，将网站的单机部署架构，改为了双机负载均衡架构，以提高整个系统的可用性。

## 方案一、自建方案

### 1.1 负载均衡：Keepalived + Nginx + HAproxy

自建负载均衡方案的主要原理是，在每一处有单点故障风险的地方（如 Web server、Rails app server、数据库等），都使用 keepalived 配置至少两个独立进程，防止其中一个服务宕机后，导致整个网站不可访问。

参考资料：

* [Setting up a High Availability Ruby on Rails environment with keepalived, nginx, HA Proxy and Thin on Debian Lenny](https://sleekd.com/general/keepalived_nginx_haproxy_thin_ruby_on_rails/)
* [Building a Highly-Available Load Balancer with Nginx and Keepalived on CentOS
](http://www.tokiwinter.com/building-a-highly-available-load-balancer-with-nginx-and-keepalived-on-centos/)
* [How To Set Up Highly Available Web Servers with Keepalived and Floating IPs on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-highly-available-web-servers-with-keepalived-and-floating-ips-on-ubuntu-14-04)

### 1.2 数据库高可用：MySQL MHA

MySQL 数据库高可用的架构，考虑到一般网站的访问量、对 down time 的要求和各种架构配置的复杂度，采用 [MySQL MHA](https://code.google.com/p/mysql-master-ha/) 比较合适。

参考资料：

* [MySQL HA方案: MHA](http://blog.csdn.net/largetalk/article/details/10006899)
* [其他 MySQL 高可用方案及其问题](https://code.google.com/p/mysql-master-ha/wiki/Other_HA_Solutions)
* [MySQL 高可用解决方案简介](http://zhaqiang.github.io/mysql/high%20availability/2014/01/13/high-availability-solutions-for-mysql/)
* [如何防止HA集群的脑裂](http://blog.chinaunix.net/uid-20726500-id-5473292.html)

### 自建方案架构图

![自建方案架构图](/images/posts/self-build-network-design-chart.jpg)

### 自建方案费用

单台 VPS：￥65元/月

最低费用：(Nginx/App Servers x 2 ＋ MySQL DBs x 2 + MHA Manager x 1) x ￥65元/月/台 = ￥325元/月

### 优点

* 部署、运维人员对系统有完整的掌控权
* Keepalived 提供了失效切换的 callback 功能，可以在服务器宕机时执行脚本，例如发送通知邮件、尝试重启服务器等

### 缺点

* 由于网络较为复杂，对部署、运维人员的能力要求较高
* 相对方案二，硬件成本也比较高

## 方案二、购买腾讯云服务方案（推荐并且已实施的部署方案）

### 2.1 [腾讯云负载均衡](https://www.qcloud.com/doc/product/214/%E6%A6%82%E8%BF%B0)

费用：

* 负载均衡实例费用 ：公网有日租：1元/天
* 负载均衡带宽费用：
  * 云服务器按带宽计费：带宽消耗使用的是云服务器已包含的公网带宽，不另外收取带宽费用；
  * 云服务器按流量计费：用户使用公网负载均衡会产生出流量，需支付对应的流量费用。

[网络计费说明](https://www.qcloud.com/doc/product/213/%E8%B4%AD%E4%B9%B0%E7%BD%91%E7%BB%9C%E5%B8%A6%E5%AE%BD)


### 2.2 腾讯云的[云数据库 CDB for MySQL](https://www.qcloud.com/product/cdb.html)

费用：

* 推荐配置：高IO型，容量25GB，内存1000MB
* 费用：112元/月/台

### 架构图

![腾讯云服务方案网络架构图](/images/posts/qcloud-service-network-design-chart.jpg)

### 最低费用总计

负载均衡实例费用30元/月/个 + 云数据库 CDB for MySQL 112元/月/台 + App Servers x 2 x 65元/月/台 = 272元/月

### 优点

* 降低系统的配置、管理复杂度
* 数据库实时双机热备，故障秒级切换。不需自建主从，自建 RAID（腾讯云数据库的产品说明）
* 费用相对自建方案便宜

### 缺点

* 腾讯云的负载均衡在检查到服务器失效后，没有提供 callback 的功能，导致在失效切换时不能进行一些自定义操作

## 结论

综合考虑费用、维护复杂度等因素，最终我们选择了方案二来实施双机负载均衡部署。

## 遇到的问题

### 问题一：图片存储（已解决）

原有的单机架构是使用 `carrierwave` gem 将图片存储在服务器本地，那在某个用户某次访问网站更新图片后，图片被存储在了服务器1上。假如后来服务器1宕机了，那图片就会丢失了。

解决办法就是使用[腾讯云对象存储服务 COS ](https://www.qcloud.com/product/cos.html)。迁移图片到腾讯云 COS 也费了点时间。我厂 CTO Rain 还为此写了支持腾讯云 COS 存储的 carrierwave gem 插件：[carrierwave-qcloud](https://github.com/rainchen/carrierwave-qcloud)。

### 问题二：计划任务（暂未解决）

使用 `capistrano` 结合 `whenever` 部署 Rails 的后台定时任务，crontab 配置只会生成在 db role 为 primary 的服务器，这里又是一个单点故障的地方。

计划任务这个单点故障的地方，因为时间关系在最终的部署方案里没有做成高可用架构。设想的方案是配合 redis 部署一个高可用的定时任务服务器。

*P.S. 本文首发于 Beansmile 官方 blog [基于腾讯云服务的 Rails 负载均衡部署方案简介](http://www.beansmile.com/blog/posts/rails-loadbalance-deployment)，一周后发布于我自己的 blog。*

**-EOF-**
