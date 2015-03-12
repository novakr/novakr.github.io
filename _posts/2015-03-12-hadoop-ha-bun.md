---
date: 2015/3/12 14:02:47 
layout: post
title: hadoop HA发展之Backup Node
categories: hadoop
tags:  hadoop
---
## hadoop HA发展之Backup Node

官方地址：
[http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Backup_Node](http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Backup_Node)

利用新版本Hadoop自身的Failover措施，配置一个Backup Node，Backup Node在内存和本地磁盘均保存了HDFS系统最新的名字空间元数据信息。如果NameNode发生故障，可用使用Backup Node中最新的元数据信息。

![](/image/hadoop-backupnode.png)

![](/image/hadoop-backupnode2.png)

* 优点

	简单方便、无需开发、配置即可使用。

	Backup Node的内存中对当前最新元数据信息进行了备份（Namespace），避免了通过NFS挂载进行备份所带来的风险。
	
	Backup Node可以直接利用内存中的元数据信息进行Checkpoint，保存到本地，与从NameNode下载元数据进行Checkpoint的方式相比效率更高。
	
	NameNode 会将元数据操作的日志记录同步到Backup Node，Backup Node会将收到的日志记录在内存中更新元数据状态，同时更新磁盘上的edits，只有当两者操作成功，整个操作才成功。这样即便NameNode上的元数据发生损坏，Backup Node的磁盘上也保存了HDFS最新的元数据，从而保证了一致性。

* 缺点

	高版本（0.21以上）才支持。
	
	许多特性还处于开发之中，例如：当NameNode无法工作时，Backup Node目前还无法直接接替NameNode提供服务，因此当前版本的Backup Node还不具有热备功能，也就是说，当NameNode发生故障，目前还只能通过重启NameNode的方式来恢复服务。
	
	Backup Node的内存中未保存Block的位置信息，仍然需要等待下面的DataNode进行上报，因此，即便在后续的版本中实现了热备，仍然需要一定的切换时间。
	
	当前版本只允许1个Backup Node连接到NameNode。

###SecondaryNameNode Checkpoint_Node Backup_Node 对比总结

![](/image/hadoop-ha.png)