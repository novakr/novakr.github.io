---
date: 2014-11-18 10:28:29 
layout: post
title: 小记mysql数据恢复过程
categories: db
tags:  mysql db
---
目前个人负责银监局标准化数据分析平台，前天晚上给客户发了一个升级包，在本地测试都OK的，升级文档写好，邮件发出去了。第二天（17号）一大早到公司那边就电话，QQ上找说程序有问题。第一感觉是不可能呀。还好目前是试运行内部几个人使用，测试阶段。不然就不好办了。然后答应一会过去看看解决解决。  
到现场看到第一现象是WEB上直接undefined,开始以为是JS兼容，浏览器版本问题。马上打开自己笔记本，跑了一下，在本地好好的。然后去看系统日志，为了方便调试，我打开了debug模式，日志刷刷出来一堆。分析了一下发现是这个问题:

	### Error querying database.  Cause: java.sql.SQLException: Table '.\yjj\middle_data' is marked as crashed and should be repaired
	### The error may exist in file [C:\apache-tomcat-7.0.42\webapps\yjj\WEB-INF\classes\cn\com\dbappsecurity\mapper\conf\MiddleDataMapper.xml]
	### The error may involve cn.com.dbappsecurity.mapper.MiddleDataMapper.selectMiddleData-Inline
	### The error occurred while setting parameters
	### SQL: select            id, gsSerialNo, orgSerialNo, adddate, `data`         from middle_data      WHERE gsSerialNo = ?                        AND orgSerialNo = ?                        AND adddate >= ?                              AND   adddate <= ?      order by id desc
	### Cause: java.sql.SQLException: Table '.\yjj\middle_data' is marked as crashed and should be repaired
	; uncategorized SQLException for SQL []; SQL state [HY000]; error code [145]; Table '.\yjj\middle_data' is marked as crashed and should be repaired; nested exception is java.sql.SQLException: Table '.\yjj\middle_data' is marked as crashed and should be repaired
		at org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean$MethodInvokingJob.executeInternal(MethodInvokingJobDetailFactoryBean.java:320)
		at org.springframework.scheduling.quartz.QuartzJobBean.execute(QuartzJobBean.java:113)
		at org.quartz.core.JobRunShell.run(JobRunShell.java:213)
		at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:557)
	Caused by: org.springframework.jdbc.UncategorizedSQLException:

middle_data' is marked as crashed and should be repaired 这个表坏了。用Navicat for MySQL客户端连接上数据库想看看这个表，也是服错。以前也设想过，这个表可能会有问题的。上面领导现在又想起新版本，所以这个事情一直放下去，没有优化。现在问题真暴露了。毕竟不是DBA度娘派上。
解决办法：

	找到mysql的安装目录的bin/myisamchk工具，在命令行中输入： 
	
	myisamchk -c -r yjj\middle_data.MYI 
	
	然后myisamchk 工具会帮助你恢复数据表的索引。

![](/image/mysql-repair.jpg) 

至此数据恢复成功。

	myisamchk --recover --quick /path/to/tblName
    myisamchk --recover /path/to/tblName
    myisamchk --safe-recover /path/to/tblName  


后来还遇到一个问题，空指针，最好得知结果是客户DB2数据表结构有变化，连接他们库的用户没查询权限，坑...


后来想想这样的问题好像不应该出现，以后要加强

 1. 数据定期定时全量增量的备份
 2. 要有数据库变更，权限变化的通知机制，不能单方面做调整。