---
title: mac环境下 mysql sql_mode 报错
date: 2018-02-14 14:58:36
tags: [MYSQL]
categories: [MYSQL]
---
## 具体出错提示：
 ```mysql
[Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
 ```
  <!-- more -->
### 查看sql_mode
 ```mysql
select @@global.sql_mode;
 ```
查询出来的值为：
 ```mysql
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
 ```
## 去掉ONLY_FULL_GROUP_BY，重新设置值。
 ```mysql
set @@global.sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
 ```
### 上面是改变了全局sql_mode，对于新建的数据库有效。对于已存在的数据库，则需要在对应的数据下执行：
 ```mysql
set sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
 ```
## 解决办法大致有两种：
一：在sql查询语句中不需要group by的字段上使用any_value()函数
这种对于已经开发了不少功能的项目不太合适，毕竟要把原来的sql都给修改一遍
二：修改`my.cnf（windows下是my.ini）`配置文件，删掉`only_full_group_by`这一项
若我们项目的mysql安装在ubuntu上面，找到这个文件打开一看，里面并没有sql_mode这一配置项，想删都没得删。
当然，还有别的办法，打开mysql命令行，执行命令
 ```mysql
select @@sql_mode;
 ```
这样就可以查出sql_mode的值，复制这个值，在`my.cnf`中添加配置项（把查询到的值删掉`only_full_group_by`这个选项，其他的都复制过去）：
 ```mysql
sql_mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION;
 ```
如果 [mysqld] 这行被注释掉的话记得要打开注释。然后重重启mysql服务
注：使用命令
 ```mysql
set sql_mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
 ```
这样可以修改一个会话中的配置项，在其他会话中是不生效的。