---
date: 2014/12/26 16:03:47 
layout: post
title: hadoop之伪分布式（Pseudo-Distributed）模式
categories: hadoop
tags:  hadoop
---

官方文档：[http://hadoop.apache.org/docs/r2.5.0/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation](http://hadoop.apache.org/docs/r2.5.0/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)

##hadoop四大模块：

* common hadoop基础模块
* hdfs hadoop文件系统
* yarn资源管理系统
* mappred 分布式离线计算框架

###修改配置文件etc/hadoop/core-site.xml
	<property>
	  <name>fs.default.name</name>
	  <value>hdfs://hadoop-zengjr.xiaoqee.com:8020</value>
	 </property>
	
	 <property>
	  	<name>hadoop.tmp.dir</name>
	  	<value>/opt/modules/hadoop-2.5.0/data/tmp</value>
	 </property>
###修改etc/hadoop/hdfs-site.xml 指定文件副本数
	 <property>
	  	<name>dfs.replication</name>
	  	<value>1</value>
	 </property>
###修改etc/hadoop/yarn-site.xml 指定文件副本数
	 <property>
	  <name>yarn.nodemanager.aux-services</name>
	  <value>mapreduce_shuffle</value>
	 </property>
###修改etc/hadoop/mapred-site.xml 指定文件副本数
创建文件cp  mapred-site.xml.template mapred-site.xml 

    <property>
	    <name>mapreduce.framework.name</name>
	    <value>yarn</value>
    </property>

以上配置完成，


格式化NameNode

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ ./bin/hdfs namenode -format

启动NameNode

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ sbin/hadoop-daemon.sh start namenode

启动DataNode

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ sbin/hadoop-daemon.sh start datanode

启动SecondaryNameNode

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ sbin/hadoop-daemon.sh start secondarynamenode
	starting secondarynamenode, logging to /opt/modules/hadoop-2.5.0/logs/hadoop-zengjr-secondarynamenode-hadoop-zengjr.xiaoqee.com.out

查看hdfs文件系统web页面
![](/image/hadoop-pseudo-distributed-web.png)

日志文件命名规则:
【框架名称-用户名-进程名-主机名-日志格式后缀】

	-rw-rw-r-- 1 zengjr zengjr   386825 Dec 16 18:10 hadoop-zengjr-datanode-hadoop-zengjr.xiaoqee.com.log
	-rw-rw-r-- 1 zengjr zengjr     3024 Dec 16 17:32 hadoop-zengjr-datanode-hadoop-zengjr.xiaoqee.com.out