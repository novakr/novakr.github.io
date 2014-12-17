---
date: 2014-12-11 17:40:29 
layout: post
title: hadoop 64位安装编译
categories: hadoop
tags:  hadoop
---
学习hadoop有一段时间了，最近一直在想做点记录，留下个脚印。可能大家都有这样的情况，学习的时候看看，听听感觉是懂了。然后过些时间，哎呀不得了，这是什么呀，我好像见过，学过。这个命令是怎么写的，得查一查度娘。这就是典型的光说不练，容易忘记呀。感觉还是写下来比较靠谱。以后就当笔记拿出来看，有记录有练习说的出做的到，这才是真正掌握。

下面只要记录一下hadoop从源码编译的过程，目前hadoop版本已经到2.6.0了，才发布不久。这里我就以**hadoop 2.5.0**这个版本来练习。hadoop在github上的地址[https://github.com/apache/hadoop](https://github.com/apache/hadoop) 下载地址如下：  

ZIP包 [https://github.com/apache/hadoop/archive/release-2.5.0.zip](https://github.com/apache/hadoop/archive/release-2.5.0.zip)  
tar包 [https://github.com/apache/hadoop/archive/release-2.5.0.tar.gz](https://github.com/apache/hadoop/archive/release-2.5.0.tar.gz)

启动虚拟机新建几个目录：

	/opt/datas #测试数据
	/opt/modules #模块
	/opt/softwares #软件
	/opt/tools #工具

使用sftp上传相关文件到虚拟机，看图：![](/image/hadoop-sftp.jpg)

文件上传成功后，解压hadoop的tar包：

	tar -zxf /opt/softwares/hadoop-2.5.0-src.tar.gz -C /opt/tools/workspace/
解压后查看一下文件大概如下：
	
	[zengjr@hadoop-zengjr hadoop-2.5.0-src]$ ls
	BUILDING.txt  hadoop-assemblies  hadoop-common-project  hadoop-hdfs-project       hadoop-maven-plugins  hadoop-project       hadoop-tools         pom.xml
	dev-support   hadoop-client      hadoop-dist            hadoop-mapreduce-project  hadoop-minicluster    hadoop-project-dist  hadoop-yarn-project
	[zengjr@hadoop-zengjr hadoop-2.5.0-src]$ 
阅读帮助文档BUILDING.txt：

	[zengjr@hadoop-zengjr hadoop-2.5.0-src]$ more BUILDING.txt 
	Build instructions for Hadoop
	
	----------------------------------------------------------------------------------
	Requirements:
	
	* Unix System
	* JDK 1.6+
	* Maven 3.0 or later
	* Findbugs 1.3.9 (if running findbugs)
	* ProtocolBuffer 2.5.0
	* CMake 2.6 or newer (if compiling native code)
	* Zlib devel (if compiling native code)
	* openssl devel ( if compiling native hadoop-pipes )
	* Internet connection for first build (to fetch all Maven and Hadoop dependencies)
	
	----------------------------------------------------------------------------------
上面写很清楚了，需要什么东西，我们就从头开始吧

##安装一些系统依赖包

	yum install autoconf automake libtool cmake
	yum install ncurses-devel
	yum install openssl-devel
	yum install lzo-devel zlib-devel gcc gcc-c++	

##安装JDK，需要1.6以上版本哦
*解压JDK*
	
	tar -zxf jdk-7u67-linux-x64.tar.gz -C /opt/modules
*设置环境变量*

	vi /etc/profile
	
	## Set JAVA HOME
	export JAVA_HOME=/opt/modules/jdk1.7.0_67
	export PATH=$PATH:$JAVA_HOME/bin		
别忘记source一下

	source /etc/profile

*验证环境*

	[zengjr@hadoop-zengjr modules]$ source /etc/profile
	[zengjr@hadoop-zengjr modules]$ echo $JAVA_HOME
	/opt/modules/jdk1.7.0_67
	[zengjr@hadoop-zengjr modules]$ java -version
	java version "1.7.0_09-icedtea"
	OpenJDK Runtime Environment (rhel-2.3.4.1.el6_3-x86_64)
	OpenJDK 64-Bit Server VM (build 23.2-b09, mixed mode)
看到上面了没有，有openJDK,这可不是我们自己安装的，删除吧，别干扰视线

*删除系统自带JDK*

	[root@hadoop-zengjr jdk1.7.0_67]# rpm -qa|grep java
	java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64
	tzdata-java-2012j-1.el6.noarch
	java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64
	[root@hadoop-zengjr jdk1.7.0_67]# rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64 tzdata-java-2012j-1.el6.noarch java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64
##安装MAVEN，版本需要3.0以上
*解压MAVEN*

	tar -zxvfapache-maven-3.0.5-bin.tar.gz -C /opt/modules

*设置环境变量*

	vi /etc/profile
	
	##Set MAVEN HOME
	export MAVEN_HOME=/opt/modules/apache-maven-3.0.5
	export PATH=$PATH:$MAVEN_HOME/bin
别忘记source一下

	source /etc/profile

*验证环境*

	[root@hadoop-zengjr apache-maven-3.0.5]# source /etc/profile
	[root@hadoop-zengjr apache-maven-3.0.5]# echo $MAVEN_HOME
	/opt/modules/apache-maven-3.0.5
	[root@hadoop-zengjr apache-maven-3.0.5]# mvn -version
	Apache Maven 3.0.5 (r01de14724cdef164cd33c7c8c2fe155faf9602da; 2013-02-19 08:51:28-0500)
	Maven home: /opt/modules/apache-maven-3.0.5
	Java version: 1.7.0_67, vendor: Oracle Corporation
	Java home: /opt/modules/jdk1.7.0_67/jre
	Default locale: en_US, platform encoding: UTF-8
	OS name: "linux", version: "2.6.32-358.el6.x86_64", arch: "amd64", family: "unix"
 
*设置Maven repository库位置*

	vi /opt/modules/apache-maven-3.0.5/settings.xml 

可设置库位置，默认位置 .m2/repository

	mkdir -p .m2/repository

##安装findbugs
*解压findbugs*

	tar -zxf findbugs-1.3.9.tar.gz -C /opt/modules/
*设置环境变量*

	vi /etc/profile

	##Set find bug
	export FINDBUGS_HOME=/opt/modules/findbugs-1.3.9
	export PATH=$PATH:$FINDBUGS_HOME/bin
别忘记source一下

	source /etc/profile

	[root@hadoop-zengjr softwares]# source /etc/profile
*验证环境*

	[root@hadoop-zengjr softwares]# findbugs -version
	1.3.9

##安装protobuf
*解压protobuf*

	tar -zxf protobuf-2.5.0.tar.gz  -C /opt/modules/
*设置环境变量*
	## Set PROTOBUF HOME
	export PROTOBUF_HOME=/opt/modules/protobuf-2.5.0
别忘记source一下

	source /etc/profile

	[root@hadoop-zengjr softwares]# source /etc/profile
*验证环境*	

	[root@hadoop-zengjr modules]# source /etc/profile
	[root@hadoop-zengjr modules]# echo $PROTOBUF_HOME
	/opt/modules/protobuf-2.5.0

*编译，安装protobuf*

	./configure
	make
	make install

##开始编译64位hadoop
可以从Building.txt中找 *命令*

	mvn package -Pdist,native -DskipTests -Dtar
经常漫长时间，下载大概jar包，最终可以看到这样的效果

	[INFO] Executed tasks
	[INFO]
	[INFO] --- maven-javadoc-plugin:2.8.1:jar (module-javadocs) @ hadoop-dist ---
	[INFO] Building jar: /opt/modules/hadoop-2.5.0-src/hadoop-dist/target/hadoop-dist-2.5.0-javadoc.jar
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Summary:
	[INFO]
	[INFO] Apache Hadoop Main ................................ SUCCESS [8:40.256s]
	[INFO] Apache Hadoop Project POM ......................... SUCCESS [1:23.948s]
	[INFO] Apache Hadoop Annotations ......................... SUCCESS [40.992s]
	[INFO] Apache Hadoop Assemblies .......................... SUCCESS [0.578s]
	[INFO] Apache Hadoop Project Dist POM .................... SUCCESS [36.725s]
	[INFO] Apache Hadoop Maven Plugins ....................... SUCCESS [48.405s]
	[INFO] Apache Hadoop MiniKDC ............................. SUCCESS [3:49.662s]
	[INFO] Apache Hadoop Auth ................................ SUCCESS [50.001s]
	[INFO] Apache Hadoop Auth Examples ....................... SUCCESS [12.831s]
	[INFO] Apache Hadoop Common .............................. SUCCESS [5:48.996s]
	[INFO] Apache Hadoop NFS ................................. SUCCESS [22.304s]
	[INFO] Apache Hadoop Common Project ...................... SUCCESS [0.117s]
	[INFO] Apache Hadoop HDFS ................................ SUCCESS [7:05.504s]
	[INFO] Apache Hadoop HttpFS .............................. SUCCESS [5:32.631s]
	[INFO] Apache Hadoop HDFS BookKeeper Journal ............. SUCCESS [1:17.592s]
	[INFO] Apache Hadoop HDFS-NFS ............................ SUCCESS [8.439s]
	[INFO] Apache Hadoop HDFS Project ........................ SUCCESS [0.047s]
	[INFO] hadoop-yarn ....................................... SUCCESS [0.110s]
	[INFO] hadoop-yarn-api ................................... SUCCESS [1:26.797s]
	[INFO] hadoop-yarn-common ................................ SUCCESS [1:20.002s]
	[INFO] hadoop-yarn-server ................................ SUCCESS [0.102s]
	[INFO] hadoop-yarn-server-common ......................... SUCCESS [37.403s]
	[INFO] hadoop-yarn-server-nodemanager .................... SUCCESS [1:35.969s]
	[INFO] hadoop-yarn-server-web-proxy ...................... SUCCESS [4.336s]
	[INFO] hadoop-yarn-server-applicationhistoryservice ...... SUCCESS [9.140s]
	[INFO] hadoop-yarn-server-resourcemanager ................ SUCCESS [28.103s]
	[INFO] hadoop-yarn-server-tests .......................... SUCCESS [1.535s]
	[INFO] hadoop-yarn-client ................................ SUCCESS [8.046s]
	[INFO] hadoop-yarn-applications .......................... SUCCESS [0.070s]
	[INFO] hadoop-yarn-applications-distributedshell ......... SUCCESS [4.270s]
	[INFO] hadoop-yarn-applications-unmanaged-am-launcher .... SUCCESS [2.525s]
	[INFO] hadoop-yarn-site .................................. SUCCESS [0.158s]
	[INFO] hadoop-yarn-project ............................... SUCCESS [8.032s]
	[INFO] hadoop-mapreduce-client ........................... SUCCESS [0.099s]
	[INFO] hadoop-mapreduce-client-core ...................... SUCCESS [46.330s]
	[INFO] hadoop-mapreduce-client-common .................... SUCCESS [26.448s]
	[INFO] hadoop-mapreduce-client-shuffle ................... SUCCESS [5.946s]
	[INFO] hadoop-mapreduce-client-app ....................... SUCCESS [17.372s]
	[INFO] hadoop-mapreduce-client-hs ........................ SUCCESS [13.102s]
	[INFO] hadoop-mapreduce-client-jobclient ................. SUCCESS [25.954s]
	[INFO] hadoop-mapreduce-client-hs-plugins ................ SUCCESS [2.420s]
	[INFO] Apache Hadoop MapReduce Examples .................. SUCCESS [9.928s]
	[INFO] hadoop-mapreduce .................................. SUCCESS [6.106s]
	[INFO] Apache Hadoop MapReduce Streaming ................. SUCCESS [15.626s]
	[INFO] Apache Hadoop Distributed Copy .................... SUCCESS [28.412s]
	[INFO] Apache Hadoop Archives ............................ SUCCESS [3.044s]
	[INFO] Apache Hadoop Rumen ............................... SUCCESS [11.177s]
	[INFO] Apache Hadoop Gridmix ............................. SUCCESS [6.493s]
	[INFO] Apache Hadoop Data Join ........................... SUCCESS [4.002s]
	[INFO] Apache Hadoop Extras .............................. SUCCESS [4.066s]
	[INFO] Apache Hadoop Pipes ............................... SUCCESS [13.545s]
	[INFO] Apache Hadoop OpenStack support ................... SUCCESS [8.434s]
	[INFO] Apache Hadoop Client .............................. SUCCESS [12.888s]
	[INFO] Apache Hadoop Mini-Cluster ........................ SUCCESS [1.606s]
	[INFO] Apache Hadoop Scheduler Load Simulator ............ SUCCESS [16.777s]
	[INFO] Apache Hadoop Tools Dist .......................... SUCCESS [6.390s]
	[INFO] Apache Hadoop Tools ............................... SUCCESS [0.040s]
	[INFO] Apache Hadoop Distribution ........................ SUCCESS [56.853s]
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 50:17.676s
	[INFO] Finished at: Sun Nov 30 11:18:20 EST 2014
	[INFO] Final Memory: 95M/241M
	[INFO] ------------------------------------------------------------------------
	[zengjr@hadoop-zengjr hadoop-2.5.0-src]$

查看一下target下的目录，这就是编译成功后的hadoop：

	[zengjr@hadoop-zengjr target]$ pwd
	/opt/modules/hadoop-2.5.0-src/hadoop-dist/target
	[zengjr@hadoop-zengjr target]$ ll
	total 390200
	drwxrwxr-x. 2 zengjr zengjr 4096 Nov 30 11:17 antrun
	-rw-rw-r--. 1 zengjr zengjr 1625 Nov 30 11:17 dist-layout-stitching.sh
	-rw-rw-r--. 1 zengjr zengjr 642 Nov 30 11:17 dist-tar-stitching.sh
	drwxrwxr-x. 9 zengjr zengjr 4096 Nov 30 11:17 hadoop-2.5.0
	-rw-rw-r--. 1 zengjr zengjr 132942024 Nov 30 11:17 hadoop-2.5.0.tar.gz
	-rw-rw-r--. 1 zengjr zengjr 2746 Nov 30 11:17 hadoop-dist-2.5.0.jar
	-rw-rw-r--. 1 zengjr zengjr 266585202 Nov 30 11:18 hadoop-dist-2.5.0-javadoc.jar
	drwxrwxr-x. 2 zengjr zengjr 4096 Nov 30 11:17 javadoc-bundle-options
	drwxrwxr-x. 2 zengjr zengjr 4096 Nov 30 11:17 maven-archiver
	drwxrwxr-x. 2 zengjr zengjr 4096 Nov 30 11:17 test-dir

一般要编译都是因为本地链接库有问题，查看一下:

	[zengjr@hadoop-zengjr native]$ pwd
	/opt/modules/hadoop-2.5.0-src/hadoop-dist/target/hadoop-2.5.0/lib/native
	[zengjr@hadoop-zengjr native]$ ll
	total 4040
	-rw-rw-r--. 1 zengjr zengjr 972242 Nov 30 11:17 libhadoop.a
	-rw-rw-r--. 1 zengjr zengjr 1487012 Nov 30 11:17 libhadooppipes.a
	lrwxrwxrwx. 1 zengjr zengjr 18 Nov 30 11:17 libhadoop.so -> libhadoop.so.1.0.0
	-rwxrwxr-x. 1 zengjr zengjr 584896 Nov 30 11:17 libhadoop.so.1.0.0
	-rw-rw-r--. 1 zengjr zengjr 582040 Nov 30 11:17 libhadooputils.a
	-rw-rw-r--. 1 zengjr zengjr 298210 Nov 30 11:17 libhdfs.a
	lrwxrwxrwx. 1 zengjr zengjr 16 Nov 30 11:17 libhdfs.so -> libhdfs.so.0.0.0
	-rwxrwxr-x. 1 zengjr zengjr 200074 Nov 30 11:17 libhdfs.so.0.0.0

用file查看一下文件，：

	[zengjr@hadoop-zengjr native]$ file libhadoop.so.1.0.0
	libhadoop.so.1.0.0: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, not stripped

ELF 64-bit 这就是64位的了.ps虽说现在hadoop中已经有64位链接文件了，还是可能会有问题的

##生成插件
	[zengjr@hadoop-zengjr hadoop-maven-plugins]$ pwd
	/opt/tools/workspace/hadoop-2.5.0-src/hadoop-maven-plugins
	[zengjr@hadoop-zengjr hadoop-maven-plugins]$ mvn install

##hadoop生成eclipse项目
	[zengjr@hadoop-zengjr hadoop-2.5.0-src]$ mvn eclipse:eclipse -DskipTests

##解压ECLIPSE，导入eclipse
[zengjr@hadoop-zengjr tools]$ tar -zxf /opt/softwares/eclipse-jee-kepler-SR1-linux-gtk-x86_64.tar.gz

