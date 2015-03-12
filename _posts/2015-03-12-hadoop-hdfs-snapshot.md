---
date: 2015/3/12 15:17:21 
layout: post
title: hadoop HDFS快照
categories: hadoop
tags:  hadoop
---
## hadoop HDFS SnapShot
官方地址：
[http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsSnapshots.html](http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsSnapshots.html)

Hadoop从2.1.0版开始提供了HDFS SnapShot的功能。一个snapshot(快照)是一个全部文件系统、或者某个目录在某一时刻的镜像。快照在下面场景下是非常有用：

* 防止用户的错误操作：管理员可以通过以滚动的方式周期性设置一个只读的快照，这样就可以在文件系统上有若干份只读快照。如果用户意外地删除了一个文件，就可以使用包含该文件的最新只读快照来进行回复。
* 备份：管理员可以根据需求来备份整个文件系统，一个目录或者单一一个文件。管理员设置一个只读快照，并使用这个快照作为整个全量备份的开始点。增量备份可以通过比较两个快照的差异来产生。
* 试验/测试: 一个用户当想要在数据集上测试一个应用程序。一般情况下，如果不做该数据集的全量拷贝，测试应用程序会覆盖/损坏原来的生产数据集，这是非常危险的。管理员可以为用户设置一个生产数据集的快照（Read write)用于用户测试使用。在快照上的改变不会影响原有数据集。
* 灾难恢复：只读快照可以被用于创建一个一致的时间点镜像用于拷贝到远程站点作灾备冗余。
通过下面命令对某一个路径（根目录/，某一目录或者文件)开启快照功能，那么该目录就成为了一个snapshottable的目录。snapshottable下存储的snapshots 最多为65535个，保存在该目录的.snapshot下

![](/image/hadoop-snapshots.jpg)

相关命令：
	
	#设置路径可以允许快照
	hdfs dfsadmin -allowSnapshot <path>
	#禁止路径快照
	hdfs dfsadmin -disallowSnapshot <path>
	#创建快照
	hdfs dfs -createSnapshot <path> [<snapshotName>]
	#删除快照
	hdfs dfs -deleteSnapshot <path> <snapshotName>
	#重命名快照
	hdfs dfs -renameSnapshot <path> <oldName> <newName>
	#得到快照列表
	hdfs lsSnapshottableDir
	#快照对比
	hdfs snapshotDiff <path> <fromSnapshot> <toSnapshot>
