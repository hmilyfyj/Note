title: Linux理论知识点 
date: 2016-03-09 16:45
tags: [Linux,折腾]
categories: Linux
---

结合上一篇[Linux命令总结](http:#b.fengbl.cn/2016/03/08/Linux-normal-direction/)，对一些细化的知识的做记录

<!-- more -->

---
# 知识点

## 重定向输出

Everything is file.

### 覆盖、追加

\>、>>

### 正常内容输出

    [me@linuxbox ~]$ ls -l /usr/bin > ls-output.txt

### 控制错误内容输出

    ls -l /bin/usr 2> ls-error.txt #输出
    ls -l /bin/usr 2> /dev/null #隐藏

### 同时输出错误、正常内容

    [me@linuxbox ~]$ ls -l /bin/usr > ls-output.txt 2>&1 #方法1
    [me@linuxbox ~]$ ls -l /bin/usr &> ls-output.txt #方法2

### | 管道线

#### 过滤器

    ls /bin /usr/bin | sort | uniq | less #排序、去重
    wc ls-output.txt #打印行，字和字节数
    head -n 5 ls-output.txt #打印文件开头部分/结尾部分
    tail -n 5 ls-output.txt
    ls /usr/bin | tee ls.txt | grep zip #从 Stdin 读取数据，并同时输出到 Stdout 和文件
    tail -f /var/log/messages #实时浏览

## 软硬连接

两者都有些类似快捷方式，但有些区别。

### 举个栗子：

    [me@linuxbox ~]$ touch f1          #创建一个测试文件f1
    [me@linuxbox ~]$ ln f1 f2          #创建f1的一个硬连接文件f2
    [me@linuxbox ~]$ ln -s f1 f3       #创建f1的一个符号连接文件f3
    [me@linuxbox ~]$ ls -li            # -i参数显示文件的inode节点信息

    total 0
    9797648 -rw-r--r--  2 oracle oinstall 0 Apr 21 08:11 f1
    9797648 -rw-r--r--  2 oracle oinstall 0 Apr 21 08:11 f2
    9797649 lrwxrwxrwx  1 oracle oinstall 2 Apr 21 08:11 f3 -> f1

从上面的结果中可以看出，硬连接文件f2与原文件f1的inode节点相同，均为9797648，然而符号连接文件的inode节点不同。

通过上面的测试可以看出：当删除原始文件f1后，硬连接f2不受影响，但是符号连接f1文件无效

### 总结
1. 删除符号连接f3,对f1,f2无影响；
2. 删除硬连接f2，对f1,f3也无影响；
3. 删除原文件f1，对硬连接f2没有影响，导致符号连接f3失效；
4. 同时删除原文件f1,硬连接f2，整个文件会真正的被删除。
5. 硬链接不能跨越物理设备， 硬链接不能关联目录，只能是文件。符号链接是文件的特殊类型，它包含一个指向 目标文件或目录的文本指针。

## Echo 展开
	#字符串展开
    [me@linuxbox ~]$ echo this is a test
    this is a test
    
    #目录展开
    [me@linuxbox ~]$ echo *
    Desktop Documents ls-output.txt Music Pictures Public Templates Videos
    [me@linuxbox ~]$ ls
    Desktop   ls-output.txt   Pictures   Templates
    echo D*
    Desktop  Documents
    [me@linuxbox ~]$ echo ~
    /home/me
    
    #算术表达式展开
    [me@linuxbox ~]$ echo [me@linuxbox ~]$((2 + 2))
    4
    
    #花括号展开
    [me@linuxbox ~]$ echo Number_{1..5}
	Number_1  Number_2  Number_3  Number_4  Number_5
	[me@linuxbox ~]$ echo Front-{A,B,C}-Back
	Front-A-Back Front-B-Back Front-C-Back
	[me@linuxbox ~]$ echo Number_{1..5}
	Number_1  Number_2  Number_3  Number_4  Number_5
	[me@linuxbox ~]$ echo {Z..A}
	Z Y X W V U T S R Q P O N M L K J I H G F E D C B A
	
	#参数展开
	[me@linuxbox ~]$ echo $USER
	me

## 占位标题

### 命令替换

[me@linuxbox ~]$ echo $(ls)
Desktop Documents ls-output.txt Music Pictures Public Templates
Videos

[me@linuxbox ~]$ ls -l $(which cp)
-rwxr-xr-x 1 root root 71516 2007-12-05 08:58 /bin/cp

### 引用

#### 双引号

文本在双引号中， shell 使用的特殊字符，除了 $，\ (反斜杠），和 `（倒引号）之外， 则失去它们的特殊含义，被当作普通字符来看待。

    [me@linuxbox ~]$ echo "$USER $((2+2)) $(cal)"
    me 4    February 2008
    Su Mo Tu We Th Fr Sa
    ....
#### 单引号
禁用全部展开

    [me@linuxbox ~]$ echo text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER
    text /home/me/ls-output.txt a b foo 4 me
    [me@linuxbox ~]$ echo "text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER"
    text ~/*.txt   {a,b} foo 4 me
    [me@linuxbox ~]$ echo 'text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER'
    text ~/*.txt  {a,b} $(echo foo) $((2+2)) $USER

## 权限

>权限由 “r”，“w”，和 “x” 来指定。

|  |  |
|--|--|
|u|	"user"的简写，意思是文件或目录的所有者。
|g|	用户组。
|o|	"others"的简写，意思是其他所有的人。
|a|	"all"的简写，是"u", "g"和“o”三者的联合。

### chmod

#### 通过数字
    [me@linuxbox ~]$ > foo.txt
    [me@linuxbox ~]$ ls -l foo.txt
    -rw-rw-r-- 1 me    me    0  2008-03-06 14:52 foo.txt
    [me@linuxbox ~]$ chmod 600 foo.txt
    [me@linuxbox ~]$ ls -l foo.txt
    -rw------- 1 me    me    0  2008-03-06 14:52 foo.txt
    
#### 通过字母改变

实例


|  |  |
|--|--|
|u+x	|为文件所有者添加可执行权限。
|u-x	|删除文件所有者的可执行权限。
|+x	|为文件所有者，用户组，和其他所有人添加可执行权限。 等价于 a+x。
|o-rw	|除了文件所有者和用户组，删除其他人的读权限和写权限。
|go=rw	|给群组的主人和任意文件拥有者的人读写权限。如果群组的主人或全局之前已经有了执行的权限，他们将被移除。
|u+x,go=rw	|给文件拥有者执行权限并给组和其他人读和执行的权限。多种设|定可以用逗号分开。

### umask

掩码：删除r、w、x属性。

默认的文件权限：0666 - 掩码
默认的文件权限：0777 - 掩码

#### 举个栗子

    [root@bfmbch ln_test]# umask 0022 #修改掩码
    [root@bfmbch ln_test]# mkdir test #创建文件夹
    [root@bfmbch ln_test]# touch test.txt #创建
    [root@bfmbch ln_test]# ll | grep test
    drwxr-xr-x 2 root root 4096 Mar 10 16:49 test
    -rw-r--r-- 1 root root    0 Mar 10 16:51 test.txt

## 更换身份

三种方式：

1. 注销系统并以其他用户身份重新登录系统。
2. 使用 su 命令。
3. 使用 sudo 命令。

### su

    su [-[l]] [user]

	#切换用户
    [me@linuxbox ~]$ su -
    Password:
    [root@linuxbox ~]#
    
    [root@linuxbox ~]# exit
    [me@linuxbox ~]$
    
    #单纯执行命令
    su -c 'command'
    
    [me@linuxbox ~]$ su -c 'ls -l /root/*'
    Password:
    -rw------- 1 root root    754 2007-08-11 03:19 /root/anaconda-ks.cfg
    
    /root/Mail:
    total 0
    [me@linuxbox ~]$

# 概念

## 符号链接

在我们到处查看时，我们可能会看到一个目录，列出像这样的一条信息：

    lrwxrwxrwx 1 root root 11 2007-08-11 07:34 libc.so.6 -> libc-2.6.so


通配符	意义
*	匹配任意多个字符（包括零个或一个）
?	匹配任意一个字符（不包括零个）
[characters]	匹配任意一个属于字符集中的字符
[!characters]	匹配任意一个不是字符集中的字符
[[:class:]]	匹配任意一个属于指定字符类中的字符






