---
date: 2015/3/12 10:40:11 
layout: post
title: hadoop HA发展之SecondaryNameNode
categories: hadoop
tags:  hadoop
---
## hadoop HA发展之SecondaryNameNode
官方地址：
[http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Secondary_NameNode](http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Secondary_NameNode)

Secondary NameNode，只要是对NameNode进行辅助作用，定期从NameNode下载edits，fsimage进行合并，并上传回NameNode.

fsimage和edits存储目录配置属性，对应dfs.namenode.name.dir和dfs.namenode.edits.dir

NameNode打开两个端口来提供RPC服务

* 一个是给client对应配置dfs.namenode.rpc-address
* 一个端口是提供给datanode对应配置dfs.namenode.servicerpc-address,如果不指定则使用dfs.namenode.rpc-address
* 其实两个端口的RPC服务没什么区别，之所以要区分主要是因为datanode和namenode通讯不会影响client和namenode的通讯，因为同一端口同时打开的句柄毕竟是有预先设定的，缺省为10个
* Namenode需要打开一个http服务，用于提供web访问以及传输edit log,checkpoint等，对应配置为dfs.namenode.http-address

Hadoop SecondaryNameNode并不是Hadoop 第二个NameNode，它不提供NameNode服务，而仅仅是NameNode的一个工具。这个工具帮助NameNode管理Metadata数据。


NameNode的HDFS文件信息（即Metadata）记录在内存中，client的文件写操作直接修改内存中的Metadata，同时也会记录到硬盘的Edits文件，这是一个Log文件。

当NameNode重启的时候，会合并硬盘上的fsimage文件和edits文件，得到完整的Metadata信息。这个fsimage文件可以看做是一个过时的Metadata信息文件（最新的Metadata修改信息在edits文件中）。

如果edits文件非常大，那么这个合并过程就非常慢，导致HDFS长时间无法启动，如果定时将edits文件合并到fsimage，那么重启NameNode就可以非常快。

SecondaryNameNode就做这个合并的工作。
![](/image/hadoop-snn2.jpg)
![](/image/hadoop-snn0.png)

从SecondaryNameNode恢复过程

![](/image/hadoop-snn1.png)