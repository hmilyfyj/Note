title: Linux理论知识点 
date: 2016-03-09 16:45
tags: [Linux,折腾]
categories: Linux
---

结合上一篇[Linux命令总结](http://b.fengbl.cn/2016/03/08/Linux-normal-direction/)，对一些细化的知识的做记录

<!-- more -->

---
# 知识点

## 重定向输出

Everything is file.

### 覆盖、追加

\>

### 正常内容输出

    $ ls -l /usr/bin > ls-output.txt

### 控制错误内容输出

    ls -l /bin/usr 2> ls-error.txt //输出
    ls -l /bin/usr 2> /dev/null //隐藏

### 同时输出错误、正常内容

    $ ls -l /bin/usr > ls-output.txt 2>&1 //方法1
    $ ls -l /bin/usr &> ls-output.txt //方法2

### | 管道线

#### 过滤器

    ls /bin /usr/bin | sort | uniq | less //排序、去重
    wc ls-output.txt //打印行，字和字节数
    head -n 5 ls-output.txt //打印文件开头部分/结尾部分
    tail -n 5 ls-output.txt
    ls /usr/bin | tee ls.txt | grep zip //从 Stdin 读取数据，并同时输出到 Stdout 和文件
    tail -f /var/log/messages //实时浏览

## 软硬连接

两者都有些类似快捷方式，但有些区别。

### 举个栗子：

    $ touch f1          #创建一个测试文件f1
    $ ln f1 f2          #创建f1的一个硬连接文件f2
    $ ln -s f1 f3       #创建f1的一个符号连接文件f3
    $ ls -li            # -i参数显示文件的inode节点信息

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




