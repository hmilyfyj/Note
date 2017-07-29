---
title: 2017-7-29-RDS-RECOVERY-ON-LOCAL-DOCKER
date:  2017-07-29 15:51:44
tags: Note
categories: Note
grammar_cjkRuby: true
---



<!-- more -->

---

## 测试环境 数据的搭建：

### 安装 备份工具
https://www.percona.com/doc/percona-xtrabackup/LATEST/installation/yum_repo.html

### 下载备份数据

``` shell
wget -c '<数据备份文件外网下载地址>' -O <自定义文件名>.tar.gz
```

参数说明：

- -c：启用断点续传模式。

- -O：将下载的结果保存为指定的文件（建议使用URL中包含的文件名）。

说明：若提示显示100%进度，则表示文件下载完成。

### 解压

``` shell
wget http://oss.aliyuncs.com/aliyunecs/rds_backup_extract.sh
tar vizxf filename.tar.gz
bash rds_backup_extract.sh -f <数据备份文件名>.tar.gz -C /home/mysql/data
```


执行如下命令，解压已下载的数据备份文件。

bash rds_backup_extract.sh -f <数据备份文件名>.tar.gz -C /home/mysql/data
参数说明：

-f：指定要解压的备份集文件。

-C：指定文件要解压到的目录。可选参数，若不指定就解压到当前目录。

### 恢复 

``` stylus
innobackupex --defaults-file=/home/mysql/data/backup-my.cnf --apply-log /home/mysql/data
```


### 注释内容必要的配置内容

``` shell
vi /home/mysql/data/backup-my.cnf
chown -R mysql:mysql /home/mysql/data
mysqld_safe --defaults-file=/home/mysql/data/backup-my.cnf --user=mysql --datadir=/home/mysql/data &
```

``` 
mysqld_safe --defaults-file=/home/mysql/data/backup-my.cnf --user=mysql --skip-grant-tables --datadir=/home/mysql/data &
```


``` stylus
#innodb_fast_checksum
#innodb_page_size
#innodb_log_block_size


#innodb_log_checksum_algorithm=innodb
#rds_encrypt_data=false
#innodb_encrypt_algorithm=aes_128_ecb
```


### 启动 mysql、phpmyadmin docker
注意：mysql 需要 5.6.0



``` shell
docker run -d -p 3306:3306 -v /root/database:/home/mysql/data --name mysql mysql:latest  mysqld_safe --defaults-file=/home/mysql/data/backup-my.cnf --user=mysql --datadir=/home/mysql/data --skip-grant-tables


```

踩坑：

``` shell
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: Yes)
```

1.使用了 localhost 链接
改为： `mysql -h 127.0.0.1 -u xxx -p`

http://www.cnblogs.com/hyzhou/archive/2011/12/06/2278236.html

2.密码错误
重置密码

``` shell
mysqld_safe  --skip-grant-tables 
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('your_new_password');
$ mysql -u root mysql
select host, user from user;
$mysql> UPDATE user SET Password=PASSWORD('my_password') where USER='root';
update user set host = '%' where user = 'root';
$mysql> FLUSH PRIVILEGES;
```

## 
https://linuxconfig.org/mysql-error-1045-28000-access-denied-for-user-root-solution
https://stackoverflow.com/questions/10299148/mysql-error-1045-28000-access-denied-for-user-billlocalhost-using-passw

//mysql docker
https://dev.aliyun.com/detail.html?spm=5176.2020520152.210.d103.322c14a0ggGFgm&repoId=1239






## 数据库升级

``` shell
mysql_upgrade
```


## 阿里云相关教程
https://help.aliyun.com/document_detail/26212.html?spm=5176.doc26206.6.725.Ru33Qh
https://www.ilanni.com/?p=10861


## 查看系统版本
http://www.linuxidc.com/Linux/2014-12/110748.htm
http://www.ha97.com/2987.html

## 本地 mysql 无法识别备份中参数
http://www.cnblogs.com/jcli/p/6253254.html