---
title: 2018-7-10-Mysql-Performance-Among-INT,TIMESTAMP,DATETIME
date: 2018-07-10 15:00:03
tags: Mysql
categories: Mysql
---

RT

<!-- more -->

---

![enter description here](https://i.imgur.com/5js6cCo.png)

>对于MyISAM引擎，不建立索引的情况下（推荐），效率从高到低：int > UNIX_TIMESTAMP(timestamp) > datetime（直接和时间比较）>timestamp（直接和时间比较）>UNIX_TIMESTAMP(datetime) 。



>对于MyISAM引擎，建立索引的情况下，效率从高到低： UNIX_TIMESTAMP(timestamp) > int > datetime（直接和时间比较）>timestamp（直接和时间比较）>UNIX_TIMESTAMP(datetime) 。



>对于InnoDB引擎，没有索引的情况下(不建议)，效率从高到低：int > UNIX_TIMESTAMP(timestamp) > datetime（直接和时间比较） > timestamp（直接和时间比较）> UNIX_TIMESTAMP(datetime)。



对于InnoDB引擎，建立索引的情况下，效率从高到低：int > datetime（直接和时间比较） > timestamp（直接和时间比较）> UNIX_TIMESTAMP(timestamp) > UNIX_TIMESTAMP(datetime)。



>一句话，对于MyISAM引擎，采用 UNIX_TIMESTAMP(timestamp) 比较；对于InnoDB引擎，建立索引，采用 int 或 datetime直接时间比较。


http://www.piaoyi.org/database/MYSQL-INT-TIMESTAMP-DATETIME.html