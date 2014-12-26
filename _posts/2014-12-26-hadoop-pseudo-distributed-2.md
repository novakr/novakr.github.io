---
date: 2014/12/26 16:03:47 
layout: post
title: hadoop之伪分布式（Pseudo-Distributed）模式2
categories: hadoop
tags:  hadoop
---

##关于HDFS文件系统的一些命令

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ ./bin/hdfs dfs
	Usage: hadoop fs [generic options]
	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
	[-chown [-R] [OWNER][:[GROUP]] PATH...]
	[-copyFromLocal [-f] [-p] <localsrc> ... <dst>]
	[-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-count [-q] <path> ...]
	[-cp [-f] [-p | -p[topax]] <src> ... <dst>]
	[-createSnapshot <snapshotDir> [<snapshotName>]]
	[-deleteSnapshot <snapshotDir> <snapshotName>]
	[-df [-h] [<path> ...]]
	[-du [-s] [-h] <path> ...]
	[-expunge]
	[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-getfacl [-R] <path>]
	[-getfattr [-R] {-n name | -d} [-e en] <path>]
	[-getmerge [-nl] <src> <localdst>]
	[-help [cmd ...]]
	[-ls [-d] [-h] [-R] [<path> ...]]
	[-mkdir [-p] <path> ...]
	[-moveFromLocal <localsrc> ... <dst>]
	[-moveToLocal <src> <localdst>]
	[-mv <src> ... <dst>]
	[-put [-f] [-p] <localsrc> ... <dst>]
	[-renameSnapshot <snapshotDir> <oldName> <newName>]
	[-rm [-f] [-r|-R] [-skipTrash] <src> ...]
	[-rmdir [--ignore-fail-on-non-empty] <dir> ...]
	[-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
	[-setfattr {-n name [-v value] | -x name} <path>]
	[-setrep [-R] [-w] <rep> <path> ...]
	[-stat [format] <path> ...]
	[-tail [-f] <file>]
	[-test -[defsz] <path>]
	[-text [-ignoreCrc] <src> ...]
	[-touchz <path> ...]
	[-usage [cmd ...]]
新建文件夹，上传文件，

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -mkdir /user
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -mkdir /user/zengjr
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -ls -R
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -ls -R /
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:01 /test
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:14 /user
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:14 /user/zengjr
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -mkdir /user/zengjr/input
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -put etc/hadoop/*.xml /user/zengjr/input
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -ls -R /
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:01 /test
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:14 /user
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:15 /user/zengjr
	drwxr-xr-x - zengjr supergroup 0 2014-11-30 23:17 /user/zengjr/input
	-rw-r--r-- 1 zengjr supergroup 3589 2014-11-30 23:17 /user/zengjr/input/capacity-scheduler.xml
	-rw-r--r-- 1 zengjr supergroup 993 2014-11-30 23:17 /user/zengjr/input/core-site.xml
	-rw-r--r-- 1 zengjr supergroup 9201 2014-11-30 23:17 /user/zengjr/input/hadoop-policy.xml
	-rw-r--r-- 1 zengjr supergroup 849 2014-11-30 23:17 /user/zengjr/input/hdfs-site.xml
	-rw-r--r-- 1 zengjr supergroup 620 2014-11-30 23:17 /user/zengjr/input/httpfs-site.xml
	-rw-r--r-- 1 zengjr supergroup 844 2014-11-30 23:17 /user/zengjr/input/mapred-site.xml
	-rw-r--r-- 1 zengjr supergroup 794 2014-11-30 23:17 /user/zengjr/input/yarn-site.xml

	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar grep /user/zengjr/input output 'dfs[a-z.]+'
	结果查看（或者文件系统中查看50070端口，50090）
	[zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -ls /user/zengjr/output
	Found 2 items
	-rw-r--r--   1 zengjr supergroup          0 2014-11-30 23:28 /user/zengjr/output/_SUCCESS
	-rw-r--r--   1 zengjr supergroup         29 2014-11-30 23:28 /user/zengjr/output/part-r-00000
	    [zengjr@hadoop-zengjr hadoop-2.5.0]$ bin/hdfs dfs -cat /user/zengjr/output/part-r-00000
	1       dfsadmin
	1       dfs.replication