---
date: 2015/3/9 15:13:02  
layout: post
title: zookeeper单机模式安装
categories: hadoop
tags:  zookeeper
---
##zookeeper单机模式安装

下载zookeeper

* [apache官网下载zookeeper](http://zookeeper.apache.org/releases.html#download)
* [github下载zookeeper](https://github.com/apache/zookeeper.git)

###单节点安装

修改配置文件

	#修改配置文件名
	mv zoo_sample.cfg zoo.cfg
	#创建数据文件夹
	[zengjr@hadoop-zengjr zookeeper]$ mkdir data
	[zengjr@hadoop-zengjr zookeeper]$ cd data/
	[zengjr@hadoop-zengjr data]$ pwd
	/opt/modules/zookeeper/data
	
	#修改配置
	# the directory where the snapshot is stored.
	# do not use /tmp for storage, /tmp here is just 
	# example sakes.
	dataDir=/opt/modules/zookeeper/data
	# the port at which the clients will connect
配置文件说明：

tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。

dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。

clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。


启动zookeeper
	
	#zkServer的用法
	[zengjr@hadoop-zengjr bin]$ ./zkServer.sh 
	JMX enabled by default
	Using config: /opt/modules/zookeeper/bin/../conf/zoo.cfg
	Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}	
	
	[zengjr@hadoop-zengjr bin]$ ./zkServer.sh start
	JMX enabled by default
	Using config: /opt/modules/zookeeper/bin/../conf/zoo.cfg
	Starting zookeeper ... STARTED
	[zengjr@hadoop-zengjr bin]$ jps
	3318 Jps
	3301 QuorumPeerMain
	1991 NameNode
	2449 NodeManager
	2093 DataNode

验证zookeeper

	[zengjr@hadoop-zengjr bin]$ ./zkServer.sh status
	JMX enabled by default
	Using config: /opt/modules/zookeeper/bin/../conf/zoo.cfg
	Mode: standalone

客户端连接zookeeper

	[zengjr@hadoop-zengjr bin]$ ./zkCli.sh -server localhost:2181
	Connecting to localhost:2181
	2014-12-31 12:30:53,094 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
	2014-12-31 12:30:53,098 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=hadoop-zengjr.xiaoqee.com
	2014-12-31 12:30:53,099 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.7.0_67
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/opt/modules/jdk1.7.0_67/jre
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/opt/modules/zookeeper/bin/../build/classes:/opt/modules/zookeeper/bin/../build/lib/*.jar:/opt/modules/zookeeper/bin/../lib/slf4j-log4j12-1.6.1.jar:/opt/modules/zookeeper/bin/../lib/slf4j-api-1.6.1.jar:/opt/modules/zookeeper/bin/../lib/netty-3.7.0.Final.jar:/opt/modules/zookeeper/bin/../lib/log4j-1.2.16.jar:/opt/modules/zookeeper/bin/../lib/jline-0.9.94.jar:/opt/modules/zookeeper/bin/../zookeeper-3.4.6.jar:/opt/modules/zookeeper/bin/../src/java/lib/*.jar:/opt/modules/zookeeper/bin/../conf:
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
	2014-12-31 12:30:53,105 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
	2014-12-31 12:30:53,106 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-358.el6.x86_64
	2014-12-31 12:30:53,106 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=zengjr
	2014-12-31 12:30:53,106 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/zengjr
	2014-12-31 12:30:53,106 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/opt/modules/zookeeper/bin
	2014-12-31 12:30:53,109 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@5b4c1313
	Welcome to ZooKeeper!
	2014-12-31 12:30:53,141 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@975] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
	2014-12-31 12:30:53,167 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@852] - Socket connection established to localhost/127.0.0.1:2181, initiating session
	JLine support is enabled
	2014-12-31 12:30:53,510 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1235] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x14aa16151c90000, negotiated timeout = 30000
	
	WATCHER::
	
	WatchedEvent state:SyncConnected type:None path:null

zkCli.sh 常用命令

	ZooKeeper -server host:port cmd args
    connect host:port
    get path [watch]
    ls path [watch]
    set path data [version]
    rmr path
    delquota [-n|-b] path
    quit 
    printwatches on|off
    create [-s] [-e] path data acl
    stat path [watch]
    close 
    ls2 path [watch]
    history 
    listquota path
    setAcl path acl
    getAcl path
    sync path
    redo cmdno
    addauth scheme auth
    delete path [version]
    setquota -n|-b val path








