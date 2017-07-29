---
title: 2017-7-29-RDS-RECOVERY-ON-LOCAL-DOCKER
date:  2017-07-29 15:51:44
tags: Note
categories: Note
grammar_cjkRuby: true
---



<!-- more -->

---


``` 
mysqld_safe --defaults-file=/home/mysql/data/backup-my.cnf --user=mysql --skip-grant-tables --datadir=/home/mysql/data &
```



``` stylus
docker run -d -p 3306:3306 -v /root/database:/home/mysql/data mysql:latest  mysqld_safe --defaults-file=/home/mysql/data/backup-my.cnf --user=mysql --datadir=/home/mysql/data --skip-grant-tables
```

踩坑：

``` shell
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: Yes)
```

1.使用了 localhost 链接
改为： `mysql -h -u xxx -p`

http://www.cnblogs.com/hyzhou/archive/2011/12/06/2278236.html

2.密码错误
重置密码

``` shell
mysqld_safe  --skip-grant-tables 
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('your_new_password');
$ mysql -u root mysql
$mysql> UPDATE user SET Password=PASSWORD('my_password') where USER='root';
$mysql> FLUSH PRIVILEGES;
```
https://linuxconfig.org/mysql-error-1045-28000-access-denied-for-user-root-solution


//mysql
https://dev.aliyun.com/detail.html?spm=5176.2020520152.210.d103.322c14a0ggGFgm&repoId=1239






## 数据库升级

``` shell
mysql_upgrade
```



https://help.aliyun.com/document_detail/26212.html?spm=5176.doc26206.6.725.Ru33Qh
https://www.ilanni.com/?p=10861
https://www.percona.com/doc/percona-xtrabackup/LATEST/installation/yum_repo.html