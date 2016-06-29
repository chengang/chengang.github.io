---
id: 4325
title: CentOS7下初始化PostgreSQL
date: 2014-08-27T17:00:16+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4325
permalink: /4325.html
categories:
  - 备忘
tags:
  - centos7
  - postgresql
---
1、安装epel；

<pre>rpm -ivh http://mirrors.hustunique.com/epel/beta/7/x86_64/epel-release-7-0.2.noarch.rpm
</pre>

或者

<pre>yum install epel-release
</pre>

2、安装postgresql；

<pre>yum install postgresql*
</pre>

3、初始化数据库；

<pre>postgresql-setup initdb
</pre>

4、启动postgresql并设置为开机自启动；

<pre>systemctl restart postgresql
systemctl enable postgresql
</pre>

5、登进数据库看看状态；（可略）

<pre>su - postgres
psql
\du （查看角色）
\l （列出所有数据库）
\q （退出）
</pre>

6、创建角色（postgresql中的用户）和数据库实例；

<pre>su - postgres
createuser dbuser
createdb -e -O dbuser dbname
</pre>

7、给新用户设定密码

<pre>su - postgres
psql
\password dbuser （输入两次密码）
vim /var/lib/pgsql/data/pg_hba.conf
</pre>

在/var/lib/pgsql/data/pg_hba.conf中，将默认验证方法

<pre>host    all             all             127.0.0.1/32            ident
</pre>

改为密码验证

<pre>host    all             all             127.0.0.1/32            md5
</pre>

8、重启数据库，让新的验证方法生效

<pre>systemctl restart postgresql
</pre>

9、新用户登录数据库；

<pre>psql -U dbuser -d dbname -h 127.0.0.1 （输入之前的密码）
</pre>

10、玩耍；（以下操作皆在postgresql终端中）

<pre>\dp （查看当前库中的表）
create table test1 (t1 int, t2 varchar(20));
\dp

\d test1 （查看表结构）

select * from test1 limit 10;

insert into test1 (t1,t2) values (11, 'ccccc'), (22, 'aaaa');

select * from test1 limit 10;

truncate table test1;

select * from test1 limit 10;

drop table test1;

select * from test1 limit 10;

\dp
</pre>