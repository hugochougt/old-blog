---
layout: post
title: "Hadoop CDH 单节点部署"
date: 2013-06-09 23:13
comments: true
categories: [hadoop, CDH]
---

在单台机器上部署 CDH 的大致步骤如下：

*   找到一台安装了支持 CDH 的 [OS](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/latest/CDH4-Requirements-and-Supported-Versions/cdhrsv_topic_1.html "Supported OS") 的机器，我是在 Ubuntu 12.04 LTS 上安装的
*   创建一个普通用户，例如 hadoop
*   安装并启动 sshd
*   设置无密码 ssh 登录 localhost
*   下载并安装 Oracle Java Development Kit (1.6 or later)
*   下载并解压 CDH 的压缩包 - [CDH Downloads](https://ccp.cloudera.com/display/SUPPORT/CDH+Downloads)，我用的是 hadoop-2.0.0-cdh4.2.0 版本
*   设置 Hadoop CDH 的配置文件
*   格式化 Hadoop namenode
*   启动 Hadoop 服务
*   检查是否部署成功

前面的安装操作系统、创建用户、设置 ssh、安装 JDK 等步骤这里不作说明，本文重点是 CDH 的相关配置。

<!-- more -->

### 1、配置环境变量
往 ~/.bashrc 文件末尾添加以下内容：
```
# Hadoop CDH env variables
export HADOOP_PREFIX=${HOME}/Programs/hadoop
export HADOOP_HOME=${HADOOP_PREFIX}
export HADOOP_MAPRED_HOME=${HADOOP_PREFIX}
export HADOOP_COMMON_HOME=${HADOOP_PREFIX}
export HADOOP_HDFS_HOME=${HADOOP_PREFIX}
export YARN_HOME=${HADOOP_PREFIX}
export HADOOP_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop
export YARN_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop
export HDFS_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop

export PATH=${HADOOP_PREFIX}/bin:${PATH}

export CLASSPATH=${JAVA_HOME}/lib:${JRE_HOME}/lib:${HADOOP_HOME}/lib:${CLASSPATH}
```

执行 `source ~/.bashrc` 使其生效，然后执行 `hadoop version` 确认环境变量生效。

P.S. 这里假设 CDH 解压在了 ${HOME}/Programs/hadoop 目录中。

### 2、配置 core-site.xml
將 ${HADOOP_CONF_DIR}/core-site.xml 文件修改为以下内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

### 3、配置 mapred-site.xml
將 ${HADOOP_CONF_DIR} 目录下的 mapred-site.xml.template 改名为 mapred-site.xml，并修改为以下内容：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

### 4、配置 hdfs-site.xml
將 ${HADOOP_CONF_DIR}/hdfs-site.xml 文件修改为以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data/hadoop/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/data/hadoop/datanode</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

然后创建 /data/hadoop/datanode 和 /data/hadoop/namenode 这两个目录。当然你也可以选择其他目录来保存 namenode 和 datanode 的数据。

### 5、配置 yarn-site.xml
將 ${HADOOP_CONF_DIR}/yarn-site.xml 文件修改为以下内容：

```xml
<?xml version="1.0"?>
<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce.shuffle</value>
    </property>
</configuration>
```

### 6、格式化 Hadoop namenode
在终端(Terminal)中执行以下命令：

``
hadoop namenode -format
```

### 7、启动 Hadoop 服务
执行 ${HADOOP_HOME}/sbin 目录下的 start-all.sh 脚本。你可能会留意到 "This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh" 这个信息，在单节点模式下可以忽略这个信息。
然后执行一下 `jps` 命令，如果配置正确，应该会有以下5个 Hadoop 的进程：

    NameNode
    SecondaryNameNode
    NodeManager
    DataNode
    ResourceManager

### 8、检查 Hadoop 能否提供正常服务
首先將本地的一个文本文件复制到 HDFS 里：

```
hadoop fs -put <local-file> /input
```

然后执行 CDH 自带的 wordcount 示例程序：

```
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /input /output
```

如无意外，终端就会显示 “INFO mapreduce.Job: Job job_XXX completed successfully” 的信息。至此，Hadoop CDH 的单节点部署算是成功了。

**-EOF-**
