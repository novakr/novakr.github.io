---
date: 2015/1/25 20:20:42 
layout: post
title: hadoop之windowns下插件安装
categories: hadoop
tags:  hadoop
---
##HADOOP windown下插件安装

先下载插件，我在安装插件过程中遇到了问题，在网上找了两个版本的，后来才发现是自己设置出问题了，反正都找过来了，这里也就放出来给大家了，下面是两个版本的，都可以使用的。

2.5.1插件下载

[hadoop-eclipse-plugin-2.5.1.jar](/doc/hadoop-eclipse-plugin-2.5.1.jar)

2.2.0插件下载：

[hadoop-eclipse-plugin-2.2.0.jar](/doc/hadoop-eclipse-plugin-2.2.0.jar)

1. 复制插件到eclipse\plugins目录中,并且重新启动eclipse，

	 	这里要说明一下，2.2.0版本插件在eclipse 版本Version: Luna Service Release 1 (4.4.1)中可用，2.5.1在这个版本的eclipse中不能用，
  如果在重新启动eclipse,在windowns==>preferences中没看到如下的图，那就果断换个版本的eclipse吧，或者换插件，不出来就是不兼容，![](/image/hadoop-eclipse-plugins-1.jpg) 


2. 设置hadoop install directory，记住一定要是安装包的目录，我就一不小心选择了源码包的目录，一直没弄好，浪费时间。
3. 设置环境变量：HADOOP_HOME目录指向安装包目录。
4. 调出![](/image/hadoop-eclipse-plugins-2.jpg) 然后新建设置![](/image/hadoop-eclipse-plugins-3.jpg)
5. 正常情况会得到这样的结果：![](/image/hadoop-eclipse-plugins-4.jpg) 这样就可以在eclipse中进行新建删除上传等操作hdfs文件系统







