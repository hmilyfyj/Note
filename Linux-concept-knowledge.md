title: Linux理论知识点 
date: 2016-03-09 16:45
tags: [Linux,折腾]
categories: Linux
---

结合上一篇[Linux命令总结](http:#b.fengbl.cn/2016/03/08/Linux-normal-direction/)，对一些细化的知识做记录

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

|文件|内容|
|---|---|
|/etc/profile	|应用于所有用户的全局配置脚本。
|~/.bash_profile	|用户私人的启动文件。可以用来扩展或重写全局配置|脚本中的设置。
|~/.bash_login	|如果文件 ~/.bash_profile 没有找到，bash 会尝试读取这个脚本。
|~/.profile	|如果文件 ~/.bash_profile 或文件 ~/.bash_login 都没有找到，bash 会试图读取这个文件。 这是基于 Debian 发行版的默认设置，比方说 Ubuntu。

非登录 shell 会话的启动文件

|文件|内容|
|---|---|
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

## 软件包管理

### 查找资源库中的软件包

软件包查找工具

|风格	|命令|
|---|---|
|Debian|	apt-get update; apt-cache search search_string|
|Red Hat|	yum search search_string|

栗子：

    yum search emacs

### 安装包

#### 通过上层工具安装

上层工具允许从一个资源库中下载一个软件包，并经过完全依赖解析来安装它。


|风格	|命令|
|---|---|
|Debian	|apt-get update; apt-get install package_name
|Red Hat	|yum install package_name

举个栗子：

    apt-get update; apt-get install emacs

#### 通过安装包

如果从某处而不是从资源库中下载了一个软件包文件，可以使用底层工具来直接（没有经过依赖解析）安装它。


|风格	|命令|
|---|---|
|Debian|	dpkg --install package_file|
|Red Hat	|rpm -i package_file|


栗子：

	rpm -i emacs-22.1-7.fc7-i386.rpm

#### 卸载


可以使用上层或者底层工具来卸载软件。下面是可用的上层工具。


|风格	|命令|
|---|---|
|Debian	apt-get |remove package_name|
|Red Hat	|yum erase package_name|

栗子：

    apt-get remove emacs


#### 经过资源库来更新软件包

最常见的软件包管理任务是保持系统中的软件包都是最新的。上层工具仅需一步就能完成 这个至关重要的任务。

|风格	|命令|
|---|---|
|Debian	|apt-get update; apt-get upgrade|
|Red Hat	|yum update|

例如：更新安装在 Debian 风格系统中的软件包：

    apt-get update; apt-get upgrade

#### 经过软件包文件来升级软件

如果已经从一个非资源库网站下载了一个软件包的最新版本，可以安装这个版本，用它来 替代先前的版本：


|风格	|命令|
|---|---|
|Debian|	dpkg --install package_file|
|Red Hat	|rpm -U package_file|

例如：把 Red Hat 系统中所安装的 emacs 的版本更新到软件包文件 emacs-22.1-7.fc7-i386.rpmz 所包含的 emacs 版本。

	rpm -U emacs-22.1-7.fc7-i386.rpm
注意：rpm 程序安装一个软件包和升级一个软件包所用的选项是不同的，而 dpkg 程序所用的选项是相同的。

#### 列出所安装的软件包

下表中的命令可以用来显示安装到系统中的所有软件包列表：


|风格	|命令|
|---|---|
|Debian	|dpkg --list|
|Red Hat	|rpm -qa|

#### 确定是否安装了一个软件包

这些底端工具可以用来显示是否安装了一个指定的软件包：


|风格	|命令|
|---|---|
|Debian	|dpkg --status package_name|
|Red Hat	|rpm -q package_name|

例如：确定是否 Debian 风格的系统中安装了这个 emacs 软件包：

	dpkg --status emacs

#### 显示所安装软件包的信息

如果知道了所安装软件包的名字，使用以下命令可以显示这个软件包的说明信息：


|风格	|命令|
|---|---|
|Debian	|apt-cache show package_name|
|Red Hat	|yum info package_name|

例如：查看 Debian 风格的系统中 emacs 软件包的说明信息：

	apt-cache show emacs

#### 查找安装了某个文件的软件包

确定哪个软件包对所安装的某个特殊文件负责，使用下表中的命令：


|风格	|命令|
|---|---|
|Debian	|dpkg --search file_name|
|Red Hat	|rpm -qf file_name|

例如：在 Red Hat 系统中，查看哪个软件包安装了/usr/bin/vim 这个文件

	rpm -qf /usr/bin/vim

## 存储媒介

暂时跳过

## 网络系统

### tracerouter

追踪跳数。

### netstat

netstat 程序被用来检查各种各样的网络设置和统计数据。通过此命令的许多选项，我们 可以看看网络设置中的各种特性。

### ftp

直接上栗子：

从匿名 FTP 服务器，其名字是 fileserver， 的/pub/_images/Ubuntu-8.04的目录下，使用 ftp 程序下载一个 Ubuntu 系统映像文件。


    [me@linuxbox ~]$ ftp fileserver
    Connected to fileserver.localdomain.
    220 (vsFTPd 2.0.1)
    Name (fileserver:me): anonymous
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> cd pub/cd\_images/Ubuntu-8.04
    250 Directory successfully changed.
    ftp> ls
    200 PORT command successful. Consider using PASV.
    150 Here comes the directory listing.
    -rw-rw-r-- 1 500 500 733079552 Apr 25 03:53 ubuntu-8.04- desktop-i386.iso
    226 Directory send OK.
    ftp> lcd Desktop
    Local directory now /home/me/Desktop
    ftp> get ubuntu-8.04-desktop-i386.iso
    local: ubuntu-8.04-desktop-i386.iso remote: ubuntu-8.04-desktop-
    i386.iso
    200 PORT command successful. Consider using PASV.
    150 Opening BINARY mode data connection for ubuntu-8.04-desktop-
    i386.iso (733079552 bytes).
    226 File send OK.
    733079552 bytes received in 68.56 secs (10441.5 kB/s)
    ftp> bye

这里是对会话期间所输入命令的解释说明：


| 命令 |意思  |
|--|--|
|ftp fileserver	|唤醒 ftp 程序，让它连接到 FTP 服务器，fileserver。
|anonymous	|登录名。输入登录名后，将出现一个密码提示。一些服务器将会接受空密码， 其它一些则会要求一个邮件地址形式的密码。如果是这种情况，试着输入 “user@example.com”。
|cd pub/cd_images/Ubuntu-8.04	|跳转到远端系统中，要下载文件所在的目录下， 注意在大多数匿名的 FTP 服务器中，支持公共下载的文件都能在目录 pub 下找到
|ls|	列出远端系统中的目录。
|lcd Desktop	|跳转到本地系统中的 ~/Desktop 目录下。在实例中，ftp 程序在工作目录 ~ 下被唤醒。 这个命令把工作目录改为 ~/Desktop
|get ubuntu-8.04-desktop-i386.iso	|告诉远端系统传送文件到本地。因为本地系统的工作目录 已经更改到了 ~/Desktop，所以文件会被下载到此目录。
|bye	|退出远端服务器，结束 ftp 程序会话。也可以使用命令 quit 和 exit。





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




