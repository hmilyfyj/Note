title: Mysql复习与总结
date: 2016-02-28 22:02
tags: [Mysql,基础不丢,在成为最厉害最厉害最厉害的路上]
categories: Mysql
---

系统的总结一下mysql相关的知识。
<!-- more -->

---

## Join 的使用

### Join

### Left Join

## NULL 值处理

- IS NULL: 当列的值是NULL,此运算符返回true。
- IS NOT NULL: 当列的值不为NULL, 运算符返回true。
- <=>: 比较操作符（不同于=运算符），当比较的的两个值为NULL时返回true。

**不能使用 = NULL 或 != NULL 在列中查找 NULL 值 。**

## 正则

查找name字段中以'st'为开头的所有数据：

    mysql> SELECT name FROM person_tbl WHERE name REGEXP '^st';

## 事务

事务的特性： 

事务有以下四个标准属性的缩写ACID，通常被称为：

- 原子性(Atomicity): 确保工作单元内的所有操作都成功完成，否则事务将被中止在故障点，和以前的操作将回滚到以前的状态。
- 一致性(Consistency): 确保数据库正确地改变状态后，成功提交的事务。
- 隔离性(1solation): 使事务操作彼此独立的和透明的。
- 可靠性(Durability): 

    mysql> begin;
    Query OK, 0 rows affected (0.00 sec)
    
    mysql> insert into dbtest values(7);
    Query OK, 1 row affected (0.00 sec)
    
    mysql> rollback;

## Alter 设计表

### 删除，添加或修改表字段


    ALTER TABLE testalter_tbl DROP i;
    ALTER TABLE testalter_tbl ADD i INT FIRST;
    ALTER TABLE testalter_tbl DROP i;
    ALTER TABLE testalter_tbl ADD i INT AFTER c;

### 修改字段类型及名称

**MODIFY** 或 **CHANGE**

把字段 c 的类型从 CHAR(1) 改为 CHAR(10)， NOT NULL 且默认值为100 

    mysql> ALTER TABLE testalter_tbl MODIFY c CHAR(10) NOT NULL DEFAULT 100;

 在 CHANGE 关键字之后，紧跟着的是你要修改的字段名，然后指定新字段的类型及名称。尝试如下实例：

    mysql> ALTER TABLE testalter_tbl CHANGE i j BIGINT;
    mysql> ALTER TABLE testalter_tbl CHANGE j j INT;

#### 修改字段默认值


你可以使用 ALTER 来修改字段的默认值，尝试以下实例：

    mysql> ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000;
    mysql> SHOW COLUMNS FROM testalter_tbl;
    +-------+---------+------+-----+---------+-------+
    | Field | Type    | Null | Key | Default | Extra |
    +-------+---------+------+-----+---------+-------+
    | c     | char(1) | YES  |     | NULL    |       |
    | i     | int(11) | YES  |     | 1000    |       |
    +-------+---------+------+-----+---------+-------+
    2 rows in set (0.00 sec)

你也可以使用 ALTER 命令及 DROP子句来删除字段的默认值，如下实例：

    mysql> ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
    mysql> SHOW COLUMNS FROM testalter_tbl;
    +-------+---------+------+-----+---------+-------+
    | Field | Type    | Null | Key | Default | Extra |
    +-------+---------+------+-----+---------+-------+
    | c     | char(1) | YES  |     | NULL    |       |
    | i     | int(11) | YES  |     | NULL    |       |
    +-------+---------+------+-----+---------+-------+
    2 rows in set (0.00 sec)
    Changing a Table Type:

### 修改数据表类型

将表 testalter_tbl 的类型修改为 MYISAM 

    mysql> ALTER TABLE testalter_tbl TYPE = MYISAM;
    mysql>  SHOW TABLE STATUS LIKE 'testalter_tbl'\G
    *************************** 1. row ****************
               Name: testalter_tbl
               Type: MyISAM
         Row_format: Fixed
               Rows: 0
     Avg_row_length: 0
        Data_length: 0
    Max_data_length: 25769803775
       Index_length: 1024
          Data_free: 0
     Auto_increment: NULL
        Create_time: 2007-06-03 08:04:36
        Update_time: 2007-06-03 08:04:36
         Check_time: NULL
     Create_options:
            Comment:
    1 row in set (0.00 sec)

### 修改表名

    mysql> ALTER TABLE testalter_tbl RENAME TO alter_tbl;

### 删除索引

## 索引

索引分**单列索引**和**组合索引**。**单列索引**，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。**组合索引**，即一个索包含多个列。

索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。

### 坏处

&ensp;&ensp;虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。建立索引会占用磁盘空间的索引文件。

### 普通索引

#### 创建索引

这是最基本的索引，它没有任何限制。它有以下几种创建方式：

    CREATE INDEX indexName ON mytable(username(length)); 

如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。
修改表结构

    ALTER mytable ADD INDEX [indexName] ON (username(length)) 

> 当用 create index 创建索引时,必须指定索引的名字，否则mysql会报错； 用 ALTER TABLE 创建索引时，可以不指定索引名字，若不指定mysql会自动生成索引名字

建立索引时，若不想用存储引擎的默认索引类型，可以指定索引的类型:

     mysql> ALTER TABLE temp_index
     ADD INDEX (first_name),
     ADD INDEX lname (last_name) USING BTREE

建表时创建：

    CREATE TABLE mytable(  
     
    ID INT NOT NULL,   
     
    username VARCHAR(16) NOT NULL,  
     
    INDEX [indexName] (username(length))  
     
    );  

删除：

    DROP INDEX indexname ON tblname
    mysql> DROP INDEX idx_actor_fname ON actor;
    mysql> ALTER TABLE actor DROP INDEX idx_actor_fname;

### 类型

    BTREE    适合连续读取数据
    RTREE    适合根据一条数据找附近的数据
    HASH      适合随机读取数据
    FULLTEXT    
    SPATIAL

### 唯一索引

唯一索引的索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

创建索引

    CREATE UNIQUE INDEX indexName ON mytable(username(length)) 

修改表结构

    ALTER mytable ADD UNIQUE [indexName] ON (username(length)) 

创建表的时候直接指定

    CREATE TABLE mytable(  
     
    ID INT NOT NULL,   
     
    username VARCHAR(16) NOT NULL,  
     
    UNIQUE [indexName] (username(length))  
     
    );   

### 显示索引信息

可以使用 SHOW INDEX 命令来列出表中的相关的索引信息。可以通过添加 \G 来格式化输出信息。

    mysql> SHOW INDEX FROM table_name\G

## 临时表

    CREATE TEMPORARY TABLE tablename

使用 SHOW TABLES命令看不到临时表。

## 复制表

### 方法1
- 使用 SHOW CREATE TABLE 命令获取创建数据表(CREATE TABLE) 语句，该语句包含了原数据表的结构，索引等。
- 复制以下命令显示的SQL语句，修改数据表名，并执行SQL语句，通过以上命令 将完全的复制数据表结构。
- 如果你想复制表的内容，你就可以使用 INSERT INTO ... SELECT 语句来实现。
### 方法2

1. 拷贝表结构到新表newadmin中。 （不会拷贝表中的数据）

    CREATE TABLE newadmin LIKE admin  

2. 拷贝数据到新表中。 注意：这个语句其实只是把select语句的结果建一个表。所以newadmin这个表不会有主键，索引。

    CREATE TABLE newadmin AS   
    (   
    SELECT *   
    FROM admin   
    )  

 
3. 真正的复制一个表。可以用下面的语句。

CREATE TABLE newadmin LIKE admin;   
INSERT INTO newadmin SELECT * FROM admin;  
 
4. 可以操作不同的数据库。

    CREATE TABLE newadmin LIKE shop.admin;   
    CREATE TABLE newshop.newadmin LIKE shop.admin;  

 
5. 也可以拷贝一个表中其中的一些字段。

    CREATE TABLE newadmin AS   
    (   
    SELECT username, password FROM admin   
    )  

 
6. 将新建的表的字段改名。

    CREATE TABLE newadmin AS   
    (   
    SELECT id, username AS uname, password AS pass FROM admin   
    )  

 
7. 拷贝一部分数据。

    CREATE TABLE newadmin AS   
    (   
    SELECT * FROM admin WHERE LEFT(username,1) = 's'   
    )  

 
8. 在创建表的同时定义表中的字段信息。

    CREATE TABLE newadmin   
    (   
    id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY   
    )   
    AS   
    (   
    SELECT * FROM admin   
    )  


## 元数据

[W3cschool](http://www.runoob.com/mysql/mysql-database-info.html)


## 序列

MySQL序列是一组整数：1, 2, 3, ...，由于一张数据表只能有一个字段自增主键， 如果你想实现其他字段也实现自动增加，就可以使用MySQL序列来实现。


[参考地址](http://www.runoob.com/mysql/mysql-using-sequences.html)

## 重复数据

[参考地址](http://www.runoob.com/mysql/mysql-handling-duplicates.html)

## 导入、导出

[enter link description here](http://www.runoob.com/mysql/mysql-database-export.html)

