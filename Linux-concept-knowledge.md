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

## sudo

sudo 命令在很多方面都相似于 su 命令，但是 sudo 还有一些非常重要的功能。管理员能够配置 sudo 命令，从而允许一个普通用户以不同的身份（通常是超级用户），通过一种非常可控的方式 来执行命令。尤其是，只有一个用户可以执行一个或多个特殊命令时，（更体现了 sudo 命令的方便性）。 s

使用 sudo 命令时，用户使用他/她自己的密码 来认证。

su 和 sudo 之间的一个重要区别是 sudo 不会重新启动一个 shell，也不会加载另一个 用户的 shell 运行环境。


    [me@linuxbox ~]$ sudo backup_script
    Password:
    System Backup Starting...
    [me@linuxbox ~]$ sudo -l
    User me may run the following commands on this host:
    (ALL) ALL


>当引进 Ubuntu 的时候，它的创作者们采取了不同的策略。默认情况下，Ubuntu 不允许用户登录 到 root 帐号（因为不能为 root 帐号设置密码），而是使用 sudo 命令授予普通用户超级用户权限。 通过 sudo 命令，最初的用户可以拥有超级用户权限，也可以授予随后的用户帐号相似的权力。

### chown

    chown [owner][:[group]] file...

#### 举个栗子

|||
|--|--|
|参数	|结果
|bob	|把文件所有者从当前属主更改为用户 bob。
|bob:users	|把文件所有者改为用户 bob，文件用户组改为用户组 users。
|:admins	|把文件用户组改为组 admins，文件所有者不变。
|bob:	|文件所有者改为用户 bob，文件用户组改为，用户 bob 登录系统时，所属的用户组。

### 改密码

passwd [user]

## 进程   

### 查看进程

#### ps
快照

#### top

实时

    top - 14:59:20 up 6:30, 2 users, load average: 0.07, 0.02, 0.00
    Tasks: 109 total,   1 running,  106 sleeping,    0 stopped,    2 zombie
    Cpu(s): 0.7%us, 1.0%sy, 0.0%ni, 98.3%id, 0.0%wa, 0.0%hi, 0.0%si
    Mem:   319496k total,   314860k used,   4636k free,   19392k buff
    Swap:  875500k total,   149128k used,   726372k free,  114676k cach
    
     PID  USER       PR   NI   VIRT   RES   SHR  S %CPU  %MEM   TIME+    COMMAND
    6244  me         39   19  31752  3124  2188  S  6.3   1.0   16:24.42 trackerd
    ....
|||
|--|--|
 | up 6:30 | 这是正常运行时间。它是计算机从上次启动到现在所运行的时间。 在这个例子里，系统已经运行了六个半小时。 |
 | 2 users | 有两个用户登录系统。 |
 | load average: | 加载平均值是指，等待运行的进程数目，也就是说，处于运行状态的进程个数， 这些进程共享 CPU。展示了三个数值，每个数值对应不同的时间周期。第一个是最后60秒的平均值， 下一个是前5分钟的平均值，最后一个是前15分钟的平均值。若平均值低于1.0，则指示计算机 工作不忙碌。 |
 | Tasks: | 总结了进程数目和各种进程状态。 |
 | Cpu(s): | 这一行描述了 CPU 正在执行的进程的特性。 |
 | 0.7%us | 0.7% of the CPU is being used for user processes. 这意味着进程在内核之外。 |
 | 1.0%sy | 1.0%的 CPU 时间被用于系统（内核）进程。 |
 | 0.0%ni | 0.0%的 CPU 时间被用于"nice"（低优先级）进程。 |
 | 98.3%id | 98.3%的 CPU 时间是空闲的。 |
 | 0.0%wa | 0.0%的 CPU 时间来等待 I/O。 |
| Mem: | 展示物理内存的使用情况。 |
| Swap: | 展示交换分区（虚拟内存）的使用情况。 |


### 控制进程

#### 退出

Ctrl-c、Ctrl-z：在前台退出。

#### 后台

    [me@linuxbox ~]$ xlogo &
    [1] 28236 #通过这条信息，shell 告诉我们，已经启动了 工作号为1（“［1］”），PID 为28236的程序。
    [me@linuxbox ~]$ ps
      PID TTY         TIME   CMD
    10603 pts/1   00:00:00   bash
    28236 pts/1   00:00:00   xlogo
    28239 pts/1   00:00:00   ps

工作控制，这个 shell 功能可以列出从终端中启动的任务。执行 jobs 命令，我们可以看到这个输出列表：

    [me@linuxbox ~]$ jobs
    [1]+ Running            xlogo &

返回到前台,一个在后台运行的进程对一切来自键盘的输入都免疫，也不能用 Ctrl-c 来中断它。使用 fg 命令，让一个进程返回前台执行：

    [me@linuxbox ~]$ jobs
    [1]+ Running        xlogo &
    [me@linuxbox ~]$ fg %1
    xlogo

fg 命令之后，跟随着一个百分号和工作序号（叫做 jobspec）。如果我们只有一个后台任务，那么 jobspec 是可有可无的。输入 Ctrl-c 来终止 xlogo 程序。

使用 fg 命令，可以恢复程序到前台运行，或者用 bg 命令把程序移到后台。

    [me@linuxbox ~]$ bg %1
    [1]+ xlogo &
    [me@linuxbox ~]$

### 杀死进程

#### kill、killalll

    kill [-signal] PID...
    killall [-u user] [-signal] name...

    [me@linuxbox ~]$ xlogo &
    [1] 28401
    [me@linuxbox ~]$ kill 28401
    [1]+ Terminated               xlogo

 kill 命令，并且指定我们想要终止的进程 PID。也可以用 jobspec（例如，“％1”）来代替 PID。

## 环境变量

    [me@linuxbox ~]$ printenv USER
    me
当使用没有带选项和参数的 set 命令时，shell 和环境变量二者都会显示，同时也会显示定义的 shell 函数。不同于 printenv 命令，set 命令的输出结果很礼貌地按照字母顺序排列：

    [me@linuxbox ~]$ set | less

    [me@linuxbox ~]$ echo $HOME
    /home/me

如果 shell 环境中的一个成员既不可用 set 命令也不可用 printenv 命令显示，则这个变量是别名。 输入不带参数的 alias 命令来查看它们:

    [me@linuxbox ~]$ alias
    alias l.='ls -d .* --color=tty'
    alias ll='ls -l --color=tty'
    alias ls='ls --color=tty'
    alias vi='vim'
    alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'

### 建立环境变量
有两种 shell 会话类型：一个是登录 shell 会话，另一个是非登录 shell 会话。

登录 shell 会话会提示用户输入用户名和密码；例如，我们启动一个虚拟控制台会话。当我们在 GUI 模式下 运行终端会话时，非登录 shell 会话会出现。

登录 shell 会话的启动文件

|||
|---|---|
|文件	内容
|/etc/profile	|应用于所有用户的全局配置脚本。
|~/.bash_profile	|用户私人的启动文件。可以用来扩展或重写全局配置|脚本中的设置。
|~/.bash_login	|如果文件 ~/.bash_profile 没有找到，bash 会尝试读取这个脚本。
|~/.profile	|如果文件 ~/.bash_profile 或文件 ~/.bash_login 都没有找到，bash 会试图读取这个文件。 这是基于 Debian 发行版的默认设置，比方说 Ubuntu。

非登录 shell 会话的启动文件

|||
|---|---|
|文件	|内容
|/etc/bash.bashrc	|应用于所有用户的全局配置文件。
|~/.bashrc	|用户私有的启动文件。可以用来扩展或重写全局配置脚本中的设置。

非登录 shell 会话也会继承它们父进程的环境设置，通常是一个登录 shell。

在普通用户看来，文件 ~/.bashrc 可能是最重要的启动文件，因为它几乎总是被读取。非登录 shell 默认 会读取它，并且大多数登录 shell 的启动文件会以能读取 ~/.bashrc 文件的方式来书写。

一个启动文件的内容：

    # .bash_profile
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
    . ~/.bashrc
    fi
    # User specific environment and startup programs
    PATH=$PATH:$HOME/bin
    export PATH

	#这个 export 命令告诉 shell 让这个 shell 的子进程可以使用 PATH 变量的内容。
    export PATH

### source

强迫 bash 重新读取修改过的 .bashrc 文件，使用下面的命令：

    [me@linuxbox ~]$ source .bashrc




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

## 进程

### 进程是怎样工作的

>当系统启动的时候，内核先把一些它自己的程序初始化为进程，然后运行一个叫做 init 的程序。init， 依次地，再运行一系列的称为 init 脚本的 shell 脚本（位于/etc），它们可以启动所有的系统服务。 其中许多系统服务以守护（daemon）程序的形式实现，守护程序仅在后台运行，没有任何用户接口。 这样，即使我们没有登录系统，至少系统也在忙于执行一些例行事务。

>一个程序可以发动另一个程序，这个事实在进程方案中，表述为一个父进程创建了一个子进程。

>内核维护每个进程的信息，以此来保持事情有序。例如，系统分配给每个进程一个数字，这个数字叫做 进程 ID 或 PID。PID 号按升序分配，init 进程的 PID 总是1。内核也对分配给每个进程的内存进行跟踪。 像文件一样，进程也有所有者和用户 ID，有效用户 ID，等等。

## 软件包管理

### .deb 与 .rpm

|  |  |
|--|--|
|包管理系统	|发行版 (部分列表)
|Debian Style (.deb)	|Debian, Ubuntu, Xandros, Linspire
|Red Hat Style (.rpm)	|Fedora, CentOS, Red Hat Enterprise |Linux, OpenSUSE, Mandriva, PCLinuxOS

### 包文件

在包管理系统中软件的基本单元是包文件。包文件是一个构成软件包的文件压缩集合。

### 上层和底层软件包工具

软件包管理系统通常由两种工具类型组成：底层工具用来处理这些任务，比方说安装和删除软件包文件， 和上层工具，完成元数据搜索和依赖解析。

||||
|---|---|---|
|发行版	|底层工具|上层工具|
|Debian-Style	|dpkg|	apt-get, aptitude
|Fedora, Red Hat Enterprise Linux, CentOS|	rpm	|yum




