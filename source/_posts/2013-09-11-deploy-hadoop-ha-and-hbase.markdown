---
layout: post
title: "部署 Hadoop HA 和 HBase 集群"
date: 2013-09-11 21:33
comments: true
categories: [Hadoop, HBase, ZooKeeper]
---

# 简介
本文主要介绍在局域网中使用 tarball 手工配置和部署 Hadoop 高可用性集群和 HBase。使用的是 cloudera 公司的 CDH 4.2.0 版本，需要下载 hadoop、HBase 和 ZooKeeper 三个压缩包。

# 集群 IP 及角色分配
 *  172.16.2.141 -- NameNode, DataNode, NodeManager, JournalNode, ZooKeeper
 *  172.16.2.142 -- NameNode, DataNode, NodeManager, JournalNode, ZooKeeper
 *  172.16.2.143 -- ResourceManager, DataNode, NodeManager, JournalNode, ZooKeeper, HRegionServer
 *  172.16.2.144 -- DataNode, NodeManager, HRegionServer
 *  172.16.2.145 -- DataNode, NodeManager, HRegionServer

# 准备工作
## 安装 ssh 并启动 sshd 服务
为集群的机器安装操作系统时（后），一定安装好 ssh 并启动 sshd 服务，不然集群无法通信。

## 创建集群的用户组和用户
分别在集群中的每台机器创建一样的用户组和用户，比如用户组和用户名均为 hadoop。可通过 `groupadd` 和 `useradd` 两个命令来创建，如下：

    # groupadd hadoop
    # useradd -g hadoop -m /home/hadoop hadoop

## ssh 无密码登录
为集群中的 hadoop 用户配置 ssh 无密码登录，具体做法请自行 google。

## 添加 IP 对主机名的映射关系
编辑 /etc/hosts 文件，为集群中的每一个 IP 添加一个对应的主机名。例如：
    172.16.2.141 node1
    172.16.2.142 node2
    172.16.2.143 node3
    172.16.2.144 node4
    172.16.2.145 node5

## 安装 JDK 1.6 (or later)
怎样安装 JDK 就不废话了，能用上 Hadoop 的，一般都有 Java 开发环境的配置经验。

## 同步整个集群的机器时间
时间同步是 HBase 要求的，可通过 `sntp` 或者 `date -s` 命令来同步整个集群的时间。时间相差太多的话 HBase 会无法启动成功的。

<!-- more -->
# 部署 ZooKeeper
本方案部署3个节点的 ZooKeeper，分别部署在 node1、node2、node3 三台机器上。ZooKeeper 部署的节点数建议为奇数个。

## 解压 ZooKeeper 压缩包
cd 进入安装目录（本文所有安装目录默认在 $HOME 下），使用下面的命令即可解压文件。解压 Hadoop 和 HBase 的 tarball 一样。

	tar -xzvf zookeeper-3.4.5-cdh4.2.0.tar.gz

## 配置 zoo.cfg
在 zookeeper-3.4.5-cdh4.2.0/conf 目录下创建 zoo.cfg 文件，并添加以下内容：

	tickTime=2000
	initLimit=5
	syncLimit=2
	dataDir=/home/hadoop/data/zookeeper/data
	dataLogDir=/home/hadoop/data/zookeeper/logs
	clientPort=2181

	server.1=172.16.2.141:2888:3888
	server.2=172.16.2.142:2888:3888
	server.3=172.16.2.143:2888:3888

然后在每台部署 ZooKeeper 服务的机器上创建 `dataDir` 和 `dataLogDir` 两个目录，并在 `dataDir` 目录中创建名为 "myid" 的文件，在 server.1 的 myid 文件中写入"1"这个数字，在 server.2 的 myid 文件写入"2"， server.3 写"3"，以此类推。

## 启动 ZooKeeper 服务
分别 ssh 登录到每台机器上，cd 进 zookeeper-3.4.5-cdh4.2.0/bin 目录，执行 `zkServer.sh start` 命令来启动 ZooKeeper 服务：

## 测试
ssh 登录到某台 ZooKeeper 的机器，在 zookeeper-3.4.5-cdh4.2.0/bin 目录，运行 `./zkCli.sh -server 172.16.2.141:2181` 命令来启动一个 ZooKeeper 客户端。在客户端输入 `help` 查看帮助，并执行一些简单操作来检查 ZooKeeper 的服务是否正常。

# 部署 Hadoop

## 解压 Hadoop 压缩包

## 配置 ~/.bashrc 文件
编辑集群中每台机器的 ~/.bashrc 文件，添加以下 Hadoop 的环境变量：

	export HADOOP_PREFIX=/home/hadoop/hadoop-2.0.0-cdh4.2.0
	export HADOOP_HOME=${HADOOP_PREFIX}
	export HADOOP_MAPRED_HOME=${HADOOP_PREFIX}
	export HADOOP_COMMON_HOME=${HADOOP_PREFIX}
	export HADOOP_HDFS_HOME=${HADOOP_PREFIX}
	export YARN_HOME=${HADOOP_PREFIX}
	export HADOOP_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop
	export YARN_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop
	export HDFS_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop

修改好后执行 `source ~/.bashcr` 命令使环境变量生效。

 **Note**: Hadoop CDH 版本的所有配置文件都在其压缩包的 etc/hadoop 目录下。

## 配置 slaves (即运行 DataNode 和 NodeManager 服务的节点)
只需要在 $HADOOP_CONF_DIR/slaves 文件中加入节点的 ip 即可。每个 ip 占一行。

    172.16.2.141
    172.16.2.142
    172.16.2.143
    172.16.2.144
    172.16.2.145

我把5台机器都作为 DataNode 和 MapReduce 的计算节点来用了。

## 修改 hadoop-env.sh
將其中的 JAVA_HOME 变量修改成 JAVA 安装目录的绝对路径，用全局环境变量的 JAVA_HOME 执行 MapReduce 应用有时会出问题。

**NOTE**:要理解下面配置中各个 properties 的意义，请先阅读 [HDFS High Availability Using the Quorum Journal Manager](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/HDFSHighAvailabilityWithQJM.html) 一文，我就不逐一讲解了。

## 配置 core-site.xml
在 core-site.xml 文件中加入以下 properties：

	<property>
	   <name>fs.defaultFS</name>
	   <value>hdfs://hadoopcluster</value>
	</property>
	<property>
	   <name>ha.zookeeper.quorum</name>
	   <value>172.16.2.141:2181，172.16.2.142:2181,172.16.2.143:2181</value>
	</property>

## 配置 hdfs-site.xml
在 hdfs-site.xml 文件中加入以下 properties:

	<property>
		 <name>dfs.namenode.name.dir</name>
		<value>/home/hadoop/data/namenode</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/home/hadoop/data/datanode</value>
	</property>
	<property>
		 <name>dfs.namenode.handler.count</name>
		 <value>100</value>
	</property>
	<property>
		<name>dfs.blocksize</name>
		<value>67108864</value>
	</property>
	<property>
		<name>dfs.nameservices</name>
		<value>hadoopcluster</value>
	</property>
	<property>
		<name>dfs.ha.namenodes.hadoopcluster</name>
		<value>nn1,nn2</value>
	</property>
	<property>
		 <name>dfs.namenode.rpc-address.hadoopcluster.nn1</name>
		 <value>172.16.2.141:8020</value>
	</property>
	<property>
		 <name>dfs.namenode.rpc-address.hadoopcluster.nn2</name>
		 <value>172.16.2.142:8020</value>
	</property>
	<property>
		 <name>dfs.namenode.shared.edits.dir</name>
		 <value>qjournal://172.16.2.141:8485;172.16.2.142:8485;172.16.2.143:8485/hadoopcluster</value>
	</property>
	<property>
		   <name>dfs.journalnode.edits.dir</name>
		   <value>/home/clouder/data/journalnode/data</value>
	</property>
	<property>
		 <name>dfs.client.failover.proxy.provider.weather</name>
		 <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	<property>
		  <name>dfs.ha.fencing.methods</name>
		  <value>shell(/home/hadoop/fencingscript.sh)</value>
	</property>
	<property>
		  <name>dfs.ha.automatic-failover.enabled</name>
		  <value>true</value>
	</property>
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>

然后在所有机器上创建 `dfs.namenode.name.dir`、`dfs.datanode.data.dir` 和 `dfs.journalnode.edits.dir` 所指定的目录。

## 创建 fencing script
將以下脚本内容保存为 fencingscript.sh 脚本，修改执行权限后，将其复制到运行 NameNode 服务的两台机器上的 $HOME 目录里（也即 hdfs-site.xml 配置的 `dfs.ha.fencing.methods` 属性所设置的目录）。

{% codeblock fencingscript.sh %}
#!/bin/bash

isNNEmpty=`jps | grep NameNode`
if [ "X${isNNEmpty}" = "X" ]; then
    ${HADOOP_HOME}/sbin/hadoop-daemon.sh start namenode
fi
exit 0
{% endcodeblock %}

这个脚本主要是在两台 NameNode 的其中一台宕机，可以自动重启宕机的 NameNode。

## 配置 mapred-site.xml
本方案使用的是 yarn 框架，所以配置如下：

	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>

	<property>
		<name>mapreduce.shuffle.port</name>
		<value>8888</value>
	</property>

## 配置 yarn-site.xml
YARN 的 ResourceManager 配置在172.16.2.143機器上，所以配置如下：

	<property>
		   <name>yarn.resourcemanager.address</name>
		   <value>172.16.2.143:8032</value>
	</property>
	<property>
		   <name>yarn.resourcemanager.scheduler.address</name>
		   <value>172.16.2.143:8030</value>
	</property>
	<property>
		   <name>yarn.resourcemanager.webapp.address</name>
		  <value>172.16.2.143:8088</value>
	</property>
	<property>
		   <name>yarn.resourcemanager.admin.address</name>
		   <value>172.16.2.143:8033</value>
	</property>
	<property>
		   <name>yarn.resourcemanager.resource-tracker.address</name>
		   <value>172.16.2.143:8031</value>
	</property>
	<property>
		   <name>yarn.nodemanager.aux-services</name>
		   <value>mapreduce.shuffle</value>
	</property>

**配置完成後，將配置文件同步到整個集群，需要進行一些初始化工作**

## 初始化 ZooKeeper
在两个NameNode中的其中一个的$HADOOP_HOME/bin目录里执行以下命令：

	$ hdfs zkfc -formatZK

觀察 Terminal 輸出，確保沒有 error。

## 啓動 JournalNode
分别在 clou01、clou02、clou03 三台机器的 $HADOOP_HOME/sbin 目录下执行以下命令来启动 JournalNode 进程：

	$ hadoop-daemon.sh start journalnode

## 格式化 NameNode
在其中一个 NameNode 的 $HADOOP_HOME/bin 目录里执行以下命令来格式化 NameNode：

	$ hadoop namenode -format

然后复制该 NameNode 的 `dfs.namenode.name.dir` 目录的数据到另外一个 NameNode 的同一目录中。

## 启动 HDFS
在任意一个 NameNode 的 sbin 目录下执行 `./start-dfs.sh` 命令来启动 HDFS。

## 启动 YARN
在 ResourceManager 角色的机器（这里是172.16.2.143）的 $HADOOP_HOME/sbin 目录下执行 `start-yarn.sh` 命令来启动 YARN。

## 测试
在浏览器地址栏中输入下表地址，可查看对应的 Hadoop 角色的状态 web 页面。

 *  172.16.2.141:50070  NameNode
 *  172.16.2.142:50070  NameNode
 *  172.16.2.141:8088   ResourceManager 

# 部署 HBase

## 解压 HBase 压缩包

## 配置 regionservers
编辑 $HBASE_HOME/conf/regionservers 文件，添加运行 region server 服务的机器 ip，每个一行。

	172.16.2.143
	172.16.2.144
	172.16.2.145

## 配置 hbase-env.sh

 1.	设置 JAVA_HOME 变量，使其为机器的 JAVA_HOME 绝对路径。
 2.	设置 HBASE_CLASSPATH，使其为 $HADOOP_CONF_DIR，因为 HBase 要找到 HDFS 的配置。
 3.	设置 HBASE_PID_DIR，使其为某本地非临时文件夹。如果保持默认的话，HBase 集群会不能正常关闭。

## 配置 hbase-site.xml
编辑 conf/hbase-site.xml 文件，添加以下属性：

	<property>
		  <name>hbase.rootdir</name>
		  <value>hdfs://hadoopcluster/hbase</value>
	</property>
	<property>
		  <name>hbase.cluster.distributed</name>
		  <value>true</value>
	</property>
	<property>
		   <name>hbase.zookeeper.quorum</name>
		   <value>172.16.2.141,172.16.2.142,172.16.2.143</value>
	</property>
	<property>
		   <name>hbase.zookeeper.property.dir</name>
		   <value>/home/hadoop/data/tmp/zookeeper</value>
	</property>
	<property>
		  <name>hbase.tmp.dir</name>
		  <value>/home/hadoop/data/hbase/tmp</value>
	</property>
	<property>
		  <name>hbase.regionserver.handler.count</name>
		  <value>100</value>
	</property>

在每台机器上创建 habse.tmp.dir 目录。

## 启动 HBase
ssh 登录到 active NameNode 所在节点，运行 `start-hbase.sh` 脚本启动 HBase 集群。

## 测试
在浏览器中输入 <active NameNode ip>:60010，查看 HBase 集群的运行状态。

**-EOF-**

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")

如果觉得本文对你有帮助，可以请作者喝[咖啡](http://me.alipay.com/zhaqiang)。
