---
date: 2015/3/12 13:52:09 
layout: post
title: hadoop HA发展之CheckPoint Node
categories: hadoop
tags:  hadoop
---
## hadoop HA发展之CheckpointNode

官方地址：
[http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Checkpoint_Node](http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Checkpoint_Node)

可能是由于Secondary NameNode这个名字给人带来的混淆，Hadoop后面的版本(1.0.4)建议不要使用Secondary NameNode，而使用CheckPoint Node。

Checkpoint Node方案与Secondary NameNode的原理基本相同，只是实现方式不同。

该方案利用Hadoop的Checkpoint机制进行备份，配置一个Checkpoint Node。该节点会定期从Primary NameNode中下载元数据信息（fsimage+edits），将edits与fsimage进行合并，在本地形成最新的Checkpoint，并上传到Primary NameNode进行更新。

当NameNode发生故障时，极端情况下（NameNode彻底无法恢复），可以在备用节点上启动一个NameNode，读取Checkpoint信息，提供服务。

* 启动方式

	bin/hdfs namenode -checkpoint

* 优点

	使用简单方便、无需开发、配置即可。

	元数据有多个备份。

* 缺点

	没有做到热备、备份节点切换时间长。
	
	Checkpoint Node所做的备份，只是最后一次Check时的元数据信息，并不是发生故障时最新的元数据信息，有可能造成数据的不一致。