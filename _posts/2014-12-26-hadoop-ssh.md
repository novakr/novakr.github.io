---
date: 2015/1/9 10:25:44 
layout: post
title: hadoop之ssh无密钥登录配置
categories: hadoop
tags:  hadoop
---
* 主要命令：

		ssh-keygen -t rsa

* 复制
	
		ssh-copy-id hadoop-zengjr.xiaoqee.com
		ssh-copy-id hadoop-zengjr
		ssh-copy-id 0.0.0.0

* 验证
* 
		ssh hadoop-zengjr.xiaoqee.com
		ssh hadoop-zengjr