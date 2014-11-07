---
date: 2014-10-23 12:44:30+00:00
layout: post
title: mongodb分片与复本集配置
thread: 164
categories: db
tags:  mongodb db
---

 很久以前就买了一本mongodb权威指南的书，看了几个章节，然后就放在书架上睡觉了。最近银监局项目上由于数据量太大（客户数据单表几亿条，10几个行的数据，以后会每天定时更新增加），现在用的mysql感觉数据查询性能方面还是有点压力。趁着之前版本提交演示完，下一次需求，bug修改还没有来之前重新看了一下monodb的书。期待用nosql数据库可以解决目前这个大数据问题。只要设想用mongod分片与复本，对大的表数据进行分片划分,做一个分布式数据库集群。  

废话不说，先看环境配置这个只是在本地测试  
	mongodb-share1-arb-10000		复本share1,仲裁节点，端口10000
	mongodb-share1-db-10001		复本share1,数据库节点，端口10001		
	mongodb-share1-db-10002 		复本share1,数据库节点，端口10002
	
	mongodb-share2-arb-20000		复本share2,仲裁节点，端口20000
	mongodb-share2-db-20001		复本share2,数据库节点，端口20001		
	mongodb-share2-db-20002 		复本share2,数据库节点，端口20002
	
	
	mongodb-mongos-30000			分片路由服务,端口30000
	
	mongodb-config-30001			分片配置服务,端口30001
	mongodb-config-30002			分片配置服务,端口30002
	mongodb-config-30003			分片配置服务,端口30003
![mongodb](/image/mongodb-rs-share-db.jpg)
![mongodb](/image/mongodb-rs-share.jpg)  
##配置复本share1##
	E:\mongodb\shareSet\mongodb-share1-db-10001\bin>
	mongod.exe --dbpath=e:\mongodb\shareSet\mongodb-share1-db-10001\data --logpath=e:\mongodb\shareSet\mongodb-share1-db-10001\log\share1-db-10001.log  --port=10001 --replSet share1/localhost:10002
	PS E:\mongodb\shareSet\mongodb-share1-db-10002\bin> 
	mongod.exe --dbpath=E:\mongodb\shareSet\mongodb-share1-db-10002\data 
	--logpath=E:\mongodb\shareSet\mongodb-share1-db-10002\log\share1-db-10002.log 
	--port=10002 --replSet share1/localhost:10001
	
	PS E:\mongodb\shareSet\mongodb-share1-arb-10000\bin> 
	.\mongod.exe --dbpath=e:\mongodb\shareSet\mongodb-share1-arb-10000\data 
	--logpath=e:\mongodb\shareSet\mongodb-share1-arb-10000\log\mongodb-share1-arb-10000.log 
	--port=10000 --replSet share1
	
	.\mongo.exe localhost:10001/admin
	db.runCommand({"replSetInitiate":{"_id":"share1","members":[{"_id":1,"host":"localhost:10001"},{"_id":2,"host":"localhost:10002"}]}})
	rs.addArb("localhost:10000")
	share1:PRIMARY> rs.config()
	{
	        "_id" : "share1",
	        "version" : 2,
	        "members" : [
	                {
	                        "_id" : 1,
	                        "host" : "localhost:10001"
	                },
	                {
	                        "_id" : 2,
	                        "host" : "localhost:10002"
	                },
	                {
	                        "_id" : 3,
	                        "host" : "localhost:10000",
	                        "arbiterOnly" : true
	                }
	        ]
	}
##配置复本share2##
	mongod.exe --dbpath=e:\mongodb\shareSet\mongodb-share2-db-20001\data --logpath=e:\mongodb\shareSet\mongodb-share2-db-20001\log\share2-db-20001.log  --port=20001 --replSet share2/localhost:20002
	
	mongod.exe --dbpath=e:\mongodb\shareSet\mongodb-share2-db-20002\data --logpath=e:\mongodb\shareSet\mongodb-share2-db-20002\log\share2-db-20002.log  --port=20002 --replSet share2/localhost:20001
	
	mongod.exe --dbpath=e:\mongodb\shareSet\mongodb-share2-arb-20000\data --logappend --logpath=e:\mongodb\shareSet\mongodb-share2-arb-20000\log\share2-arb-20000.log --port=20000 --replSet share2
	
	
	.\mongo.exe localhost:20001/admin
	db.runCommand({"replSetInitiate":{"_id":"share2","members":[{"_id":1,"host":"localhost:20001"},{"_id":2,"host":"localhost:20002"}]}})
	rs.addArb("localhost:20000")
	
	share2:PRIMARY> rs.config()
	{
	        "_id" : "share2",
	        "version" : 2,
	        "members" : [
	                {
	                        "_id" : 1,
	                        "host" : "localhost:20001"
	                },
	                {
	                        "_id" : 2,
	                        "host" : "localhost:20002"
	                },
	                {
	                        "_id" : 3,
	                        "host" : "localhost:20000",
	                        "arbiterOnly" : true
	                }
	        ]
	}
##mongodb config 启动##
	.\mongod.exe --dbpath=E:\mongodb\shareSet\mongodb-config-30001\data --logpath=E:\mongodb\shareSet\mongodb-config-30001\log\config-30001.log --port=30001
	.\mongod.exe --dbpath=E:\mongodb\shareSet\mongodb-config-30002\data --logpath=E:\mongodb\shareSet\mongodb-config-30002\log\config-30002.log --port=30002
	.\mongod.exe --dbpath=E:\mongodb\shareSet\mongodb-config-30003\data --logpath=E:\mongodb\shareSet\mongodb-config-30003\log\config-30003.log --port=30003
	#mongos 启动 configdb必须是单数
	.\mongos.exe --configdb "localhost:30001,localhost:30002,localhost:30003" --port 30000 --logpath E:\mongodb\shareSet\mongodb-mongos-30000\log\mongos-30000.log 
##添加分片##
	.\mongo.exe localhost:30000/admin
	
	sh.addShard("share1/localhost:10001,localhost:10002,localhost:10000")
	sh.addShard("share2/localhost:20001,localhost:20002,localhost:20000")
	mongos> use admin
	switched to db admin
	mongos> db.runCommand({"enablesharding":"yjj"})
	db.runCommand({"shardcollection":"yjj.indexData","key":{"schema":1,"orgno":1}})
	
	share1:SECONDARY> rs.status()
	{
	        "set" : "share1",
	        "date" : ISODate("2014-10-14T08:52:44Z"),
	        "myState" : 2,
	        "syncingTo" : "localhost:10002",
	        "members" : [
	                {
	                        "_id" : 1,
	                        "name" : "localhost:10001",
	                        "health" : 1,
	                        "state" : 2,
	                        "stateStr" : "SECONDARY",
	                        "uptime" : 1101,
	                        "optime" : Timestamp(1413274493, 1935),
	                        "optimeDate" : ISODate("2014-10-14T08:14:53Z"),
	                        "self" : true
	                },
	                {
	                        "_id" : 2,
	                        "name" : "localhost:10002",
	                        "health" : 1,
	                        "state" : 1,
	                        "stateStr" : "PRIMARY",
	                        "uptime" : 1099,
	                        "optime" : Timestamp(1413274493, 1935),
	                        "optimeDate" : ISODate("2014-10-14T08:14:53Z"),
	                        "lastHeartbeat" : ISODate("2014-10-14T08:52:43Z"),
	                        "lastHeartbeatRecv" : ISODate("2014-10-14T08:52:44Z"),
	                        "pingMs" : 0,
	                        "electionTime" : Timestamp(1413275671, 1),
	                        "electionDate" : ISODate("2014-10-14T08:34:31Z")
	                },
	                {
	                        "_id" : 3,
	                        "name" : "localhost:10000",
	                        "health" : 1,
	                        "state" : 7,
	                        "stateStr" : "ARBITER",
	                        "uptime" : 24,
	                        "lastHeartbeat" : ISODate("2014-10-14T08:52:43Z"),
	                        "lastHeartbeatRecv" : ISODate("2014-10-14T08:52:43Z"),
	                        "pingMs" : 44
	                }
	        ],
	        "ok" : 1
	}
	share1:SECONDARY>