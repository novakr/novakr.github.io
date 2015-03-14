---
date: 2015/3/14 23:51:48  
layout: post
title: spark集群安装布署
categories: spark
tags:  spark
---
## spark集群安装布署


1. JDK安装配置
* ssh无密码登录
* hadoop集群安装配置
* scala安装配置
* spark安装配置

前三步都都已经设置过了，现在只要进行scala和spark的安装

###scala安装 

[下载scala](http://www.scala-lang.org/files/archive/)

安装验证如下

	[zengjr@hadoop-zengjr spark]$ ls
	scala-2.10.4.tgz  scala-docs-2.10.4.tgz
	[zengjr@hadoop-zengjr spark]$ tar zxvf scala-2.10.4.tgz -C /opt/modules/	

	##scala
	export SCALA_HOME=/opt/modules/scala-2.10.4
	export PATH=$PATH:$SCALA_HOME/bin
	"/etc/profile" 98L, 2252C written                                                                                                                    
	[root@hadoop-zengjr scala-2.10.4]# exit
	exit
	[zengjr@hadoop-zengjr scala-2.10.4]$ source /etc/profile
	[zengjr@hadoop-zengjr scala-2.10.4]$ scala -version
	Scala code runner version 2.10.4 -- Copyright 2002-2013, LAMP/EPFL

分发内容到其它机器，修改其它机器环境

	scp -r scala-2.10.4/ zengjr@hadoop-slave01.xiaoqee.com:/opt/modules/
	scp -r scala-2.10.4/ zengjr@hadoop-slave02.xiaoqee.com:/opt/modules/

###spark安装
[下载spark](http://mirrors.hust.edu.cn/apache/spark/spark-1.2.1/spark-1.2.1.tgz)

	scp -r spark-1.2.1/ zengjr@hadoop-slave01.xiaoqee.com:/opt/modules/
	scp -r spark-1.2.1/ zengjr@hadoop-slave02.xiaoqee.com:/opt/modules/

	[zengjr@hadoop-zengjr modules]$ jps
	5267 JournalNode
	4751 QuorumPeerMain
	5452 DataNode
	5964 DFSZKFailoverController
	8138 NodeManager
	6122 NameNode
	8313 Jps

###spark编译
文档：
[http://spark.apache.org/docs/1.2.1/building-spark.html](http://spark.apache.org/docs/1.2.1/building-spark.html)

编译

	export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"	

	mvn  -Phadoop-2.4 -Dhadoop.version=2.5.0 -Dprotobuf.version=2.5.0 -Pyarn -Phive -Phive-0.13.1  -Pspark-ganglia-lgpl  -Phive-thriftserver  -DskipTests package  

成功后结果：

	Finished in 1 ms
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Summary:
	[INFO] 
	[INFO] Spark Project Parent POM .......................... SUCCESS [6:31.580s]
	[INFO] Spark Project Networking .......................... SUCCESS [4:02.924s]
	[INFO] Spark Project Shuffle Streaming Service ........... SUCCESS [13.186s]
	[INFO] Spark Project Core ................................ SUCCESS [12:32.042s]
	[INFO] Spark Project Bagel ............................... SUCCESS [58.480s]
	[INFO] Spark Project GraphX .............................. SUCCESS [3:12.488s]
	[INFO] Spark Project Streaming ........................... SUCCESS [3:32.578s]
	[INFO] Spark Project Catalyst ............................ SUCCESS [3:45.477s]
	[INFO] Spark Project SQL ................................. SUCCESS [4:15.451s]
	[INFO] Spark Project ML Library .......................... SUCCESS [5:29.892s]
	[INFO] Spark Project Tools ............................... SUCCESS [30.299s]
	[INFO] Spark Project Hive ................................ SUCCESS [6:03.313s]
	[INFO] Spark Project REPL ................................ SUCCESS [1:46.258s]
	[INFO] Spark Project YARN Parent POM ..................... SUCCESS [9.942s]
	[INFO] Spark Project YARN Stable API ..................... SUCCESS [1:43.722s]
	[INFO] Spark Project Hive Thrift Server .................. SUCCESS [1:45.940s]
	[INFO] Spark Ganglia Integration ......................... SUCCESS [18.260s]
	[INFO] Spark Project Assembly ............................ SUCCESS [1:20.399s]
	[INFO] Spark Project External Twitter .................... SUCCESS [45.683s]
	[INFO] Spark Project External Flume Sink ................. SUCCESS [1:13.772s]
	[INFO] Spark Project External Flume ...................... SUCCESS [1:00.997s]
	[INFO] Spark Project External MQTT ....................... SUCCESS [13:44.861s]
	[INFO] Spark Project External ZeroMQ ..................... SUCCESS [44.883s]
	[INFO] Spark Project External Kafka ...................... SUCCESS [1:13.872s]
	[INFO] Spark Project Examples ............................ SUCCESS [4:39.398s]
	[INFO] Spark Project YARN Shuffle Service ................ SUCCESS [10.114s]
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 1:21:48.551s
	[INFO] Finished at: Sat Feb 14 16:04:13 EST 2015
	[INFO] Final Memory: 92M/583M
	[INFO] ------------------------------------------------------------------------
	[zengjr@hadoop-zengjr spark-1.2.1]$ 	
	