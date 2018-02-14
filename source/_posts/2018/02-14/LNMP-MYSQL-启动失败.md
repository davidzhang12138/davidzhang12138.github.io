---
title: LNMP MYSQL 启动失败
date: 2018-02-14 14:31:10
tags: [MYSQL,LNMP]
categories: [MYSQL,LNMP]
---

 ## 发现
 公司测试服务器，最近频繁出现站点访问失败问题，站点经常显示空白页面 
 <!-- more -->
 
 ## 排查
 首先就猜想是不是数据库原因，所以讲自己本地项目数据库链接改为测试服务器，发现果然本地站点也出现无法打开问题
 手动重启数据库`lnmp mysql restart`，发现卡在重启过程中 无法成功，然后，去测试服务器查找mysql的error日志，
 发现在mysql目录下并没有错误文件 打开`/etc/my.cnf` 发现没有配置`log_error` 然后增加如下配置
 ```nginx
 log_error=/usr/local/mysql/error.log
 ```
接着重新重启数据库`lnmp mysql restart`，结果显而易见 还是失败，但是这次生成了 error 日志，打开日志，发现如下信息
 ```nginx
InnoDB: Completed initialization of buffer pool
InnoDB: Error: log file .\\ib_logfile0 is of different size 0 5242880 bytes
InnoDB: than specified in the .cnf file 0 10485760 bytes!
[ERROR] Plugin 'InnoDB' init function returned error.
[ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
[ERROR] Unknown/unsupported storage engine: INNODB
[ERROR] Aborting
 ```
 ## 解决
经过google 发现一个解决方案 
在`/etc/my.cnf`中 增加如下配置：
 ```nginx
innodb_fast_shutdown=0
innodb_log_file_size=5M
 ```
 然后删除`/usr/local/mysql/var` 文件夹下ib_logfile开头的文件
 重启mysql`lnmp mysql restart` 发现果然成功

## 后记

几天后发现 还是会出现问题 站点打不开
删除ib_logfile开头的文件后  发现再次生成的文件大小为0
猜测是不是存储空间不够 使用 `df -h` 命令查看 发现 果然内存不足 40G空间基本都用完了。。
然后使用`du -sh *` 查找到之前压力测试站点生成的mongo数据库并没有删除 足足占用了 17G的内存，果断删除 并且关闭压力测试 站点入口 ，
然后重启mysql `lnmp mysql restart` 后面果然运行正常 