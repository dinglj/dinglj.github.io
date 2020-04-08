---
layout : post
title : "mysql安装"
category : 规范
duoshuo: true
date : 2020-03-30
tags : [mysql,安装]
---
#下载rpm源,安装mysql

wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm 
rpm -ivh mysql57-community-release-el7-8.noarch.rpm 
yum -y install mysql-server 
#启动mysql
#service mysqld start 
systemctl start mysqld
#查看mysql状态
systemctl  status mysqld

#查看mysql状态
systemctl  stop mysqld

#查看mysql安装密码
cat /var/log/mysqld.log | grep password

# mysql 连接
mysql -u root -p

# 执行修改密码语句即可成功

alter user 'root'@'localhost' identified by '123456';

update  mysql.user set  host='%' where user='root';

flush privileges;
