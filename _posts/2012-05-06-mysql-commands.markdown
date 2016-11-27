---
layout: post
title: "mysql 常用命令小结"
date: 2012-05-06 15:32
categories: [DB]
tags: [Mysql]
---

查询
----
* 查询存在的数据库

 >show databases;

 >选择所要的某个数据库  use databasename;

* 查询当前数据库存在的所有的表

 >show tables;

* 查询表结构

 >describe tablename;

* 查询表格列的属性

 >show columns from tableName；

* 查询记录

 >select name from tablename where id=xxx;

* 查询当前时间

 >select now();

 >select current\_time;

* 查询当前日期

 >select current\_date;

* 查询当前用户

 >select user();

* 查询数据库版本

 >select version();

* 查询当前使用的数据库

 >select database();

* 查询当前服务器支持哪个存储引擎

 >show engines;

创建
----
* 创建数据库

 >create database databasename;

 >create database DATABASE\_NAME default character set utf8;

* 创建一张表

 >create table tablename (name VARCHAR(20), sex CHAR(1));

 >create table if not exists students(……);                            //创建表是先判断表是否存在

* 创建临时表：(建立临时表linshi)

 >create temporary table linshi(name varchar(10));

* 从已经有的表table1中复制表的结构到表table2

 >create table table2 select \* from table1 where 1\<\>1;              //只复制表结构

 >create table table2 select \* from table1;                           //复制表结构和表中的数据

* 往表中links加入记录

 >insert into links(name,url) values('xiaoxiaozi','www.xiaoxiaozi.com');

 >insert into links set name='xiaoxiaozi',url='www.xiaoxiaozi.com';

* copy table datas from other table

 >insert into table column1, column2 select column1 column2 from table2 where query;

修改
----
* 对表重新命名

 > alter table tablename1 rename as tablename2;

* 修改列的类型

 > alter table tablename modify id int unsigned;            //修改列id的类型为int unsigned

 > alter table tablename change id sid int unsigned;        //修改列id的名字为sid，而且把属性修改为int unsigned

* 更新表中数据

 > update tablename set sex='f' where name='john';

删除
----
* 删除某个数据库

 > drop database databasename;        //删除数据库前，没有提示

 > mysqladmin drop databasename;      //删除数据库前，有提示

* 删除某张表

 > drop table tablename;

* 清空某张表

 > delete from tablename;

* 删除符合条件的某些记录

 > delete from tablename where id=xxx;

* 删除授权：

 > revoke all privileges on \*\.\* from root@”%”;

 > delete from user where user=”root” and host=”%”;

 > flush privileges;             //刷新数据库

备份
----
* 备份数据库：(将数据库test备份)

 > mysqldump -u root -p test>c:\test.txt

* 备份表格：(备份test数据库下的mytable表格)

 > mysqldump -u root -p test mytable>c:\test.txt

* 将备份数据导入到数据库：(导回test数据库)

 > mysql -u root -p test<c:\test.txt
