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




## mysql 定时备份
1. 备份脚本: mysql_bak_script.sh
#!/bin/bash

#保存备份个数，备份7天数据
number=7
#备份保存路径
backup_dir=/mysqldata/mysqlbackup
#日期
dd=`date +%Y-%m-%d-%H-%M-%S`
#备份工具
tool=mysqldump
#用户名
username=root
#密码
password=yh2018b2b
#将要备份的数据库
database_name=csx_b2b_crm

#如果文件夹不存在则创建
if [ ! -d $backup_dir ];
then
mkdir -p $backup_dir;
fi

#记录master log postion start信息
echo "master log postion start log: $backup_dir/$database_name-$dd.sql" >> $backup_dir/log.txt
mysql -u $username -p$password -e "show master status;" >> $backup_dir/log.txt

#简单写法 mysqldump -u root -p123456 users > /root/mysqlbackup/users-$filename.sql
$tool -u $username -p$password $database_name > $backup_dir/$database_name-$dd.sql
#$tool --defaults-extra-file= /etc/my.cnf $database_name > $backup_dir/$database_name-$dd.sql

#记录master log postion end信息
echo "master log postion end log: $backup_dir/$database_name-$dd.sql" >> $backup_dir/log.txt
mysql -u $username -p$password -e "show master status;" >> $backup_dir/log.txt

#写创建备份日志
echo "create $backup_dir/$database_name-$dd.sql" >> $backup_dir/log.txt

#找出需要删除的备份
delfile=`ls -l -crt $backup_dir/*.sql | awk '{print $9 }' | head -1`

#判断现在的备份数量是否大于$number
count=`ls -l -crt $backup_dir/*.sql | awk '{print $9 }' | wc -l`

if [ $count -gt $number ]
then
#删除最早生成的备份，只保留number数量的备份
rm -rf $delfile
#写删除文件日志
echo "delete $delfile" >> $backup_dir/log.txt
fi

2. 编辑 mysql_bak_script.cron
crontab -e
添加内容:
0 2 * * * /app/backup/mysql_bak_script.sh
查看结果:
crontab -l
