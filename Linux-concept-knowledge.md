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

### wget


## 文件查找

### locate - 查找文件的简单方法

    [me@linuxbox ~]$ locate bin/zip #locate 命令将会搜索它的路径名数据库，输出任一个包含字符串“bin/zip”的路径名：
    /usr/bin/zip
    /usr/bin/zipcloak
    /usr/bin/zipgrep
    /usr/bin/zipinfo
    /usr/bin/zipnote
    /usr/bin/zipsplit

    [me@linuxbox ~]$ locate zip | grep bin
    /bin/bunzip2
    /bin/bzip2
    /bin/bzip2recover
    /bin/gunzip
    /bin/gzip
    /usr/bin/funzip
    /usr/bin/gpg-zip
    /usr/bin/preunzip
    /usr/bin/prezip
    /usr/bin/prezip-bin
    /usr/bin/unzip
    /usr/bin/unzipsfx
    /usr/bin/zip
    /usr/bin/zipcloak
    /usr/bin/zipgrep
    /usr/bin/zipinfo
    /usr/bin/zipnote
    /usr/bin/zipsplit

### find - 查找文件的复杂方式

locate 程序只能依据文件名来查找文件。

find 命令的最简单使用是，搜索一个或多个目录。如：

    [me@linuxbox ~]$ find ~

	#对于最活跃的用户帐号，这将产生一张很大的列表。因为这张列表被发送到标准输出， 我们可以把这个列表管道到其它的程序中。让我们使用 wc 程序来计算出文件的数量
    [me@linuxbox ~]$ find ~ | wc -l
47068

	#带条件搜索
	[me@linuxbox ~]$ find ~ -type d | wc -l
    1695
    [me@linuxbox ~]$ find ~ -type f | wc -l
    38737

	#根据文件大小和文件名来搜索：让我们查找所有文件名匹配 通配符模式“*.JPG”和文件大小大于1M 的文件
    [me@linuxbox ~]$ find ~ -type f -name "\*.JPG" -size +1M | wc -l
    840

	#带条件 ( expression 1 ) -or ( expression 2 )
	[me@linuxbox ~]$ find ~ \( -type f -not -perm 0600 \) -or \( -type d -not -perm 0700 \) #权限不是0600的文件和权限不是0700的目录

	#带预定义操作
	find ~ -type f -name '*.BAK' -delete #用户家目录（和它的子目录）下搜索每个以.BAK 结尾的文件名。当找到后，就删除它们。
	find ~ -print -and -type f -and -name '*.BAK' #等价于上面的命令

	#带用户定义的行为 -exec rm '{}' ';'
	find ~ -type f -name 'foo*' -ok ls -l '{}' ';' #{}代表匹配的结果。交互式地执行一个用户定义的行为。通过使用 -ok 行为来代替 -exec
	**需要将{}用单引号、双引号或反斜杠，否则不认识。bash可以不用。建议加上。**

	#提高效率：命令只执行一次
	#方法1
	find ~ -type f -name 'foo*' -exec ls -l '{}' + #;号改为+号
    -rwxr-xr-x 1 me     me 224 2007-10-29 18:44 /home/me/bin/foo
    -rw-r--r-- 1 me     me 0 2008-09-19 12:53 /home/me/foo.txt
	
	#方法2 xargs
	find ~ -type f -name 'foo\*' -print | xargs ls -l # find 命令的输出被管道到 xargs 命令，反过来，xargs 会为 ls 命令构建 参数列表，然后执行 ls 命令。
	-rwxr-xr-x 1 me     me 224 2007-10-29 18:44 /home/me/bin/foo
	-rw-r--r-- 1 me     me 0 2008-09-19 12:53 /home/me/foo.txt

find 文件类型

|文件类型	|描述|
|---|---|
|b	|块设备文件
|c	|字符设备文件
|d	|目录
|f	|普通文件
|l	|符号链接

find 大小单位

|字符|	单位|
|---|---|
|b	|512 个字节块。如果没有指定单位，则这是默认值。
|c	|字节
|w	|两个字节的字
|k	|千字节(1024个字节单位)
|M	|兆字节(1048576个字节单位)
|G	|千兆字节(1073741824个字节单位)


预定义的 find 命令操作
|操作|描述|
|---|---|
|-delete	|删除当前匹配的文件。|
|-ls	|对匹配的文件执行等同的 ls -dils 命令。并将结果发送到标准输出。|
|-print	|把匹配文件的全路径名输送到标准输出。如果没有指定其它操作，这是 默认操作。|
|-quit	|一旦找到一个匹配，退出。|

## 归档备份

### gzip、gunzip

执行后会替换源文件，压缩文件与原始文件有着相同的权限和时间戳。

栗子：

    [me@linuxbox ~]$ ls -l /etc > foo.txt
    [me@linuxbox ~]$ ls -l foo.*
    -rw-r--r-- 1 me     me 15738 2008-10-14 07:15 foo.txt
    [me@linuxbox ~]$ gzip foo.txt
    [me@linuxbox ~]$ ls -l foo.*
    -rw-r--r-- 1 me     me 3230 2008-10-14 07:15 foo.txt.gz
    [me@linuxbox ~]$ gunzip foo.txt.gz
    [me@linuxbox ~]$ ls -l foo.*
    -rw-r--r-- 1 me     me 15738 2008-10-14 07:15 foo.txt

	[me@linuxbox ~]$ zcat foo.txt.gz | less #cat 查看解压后文件的内容

参数

|  选项| 说明 |
|--|--|
|-c	|把输出写入到标准输出，并且保留原始文件。也有可能用--stdout 和--to-stdout 选项来指定。
|-d	|解压缩。正如 gunzip 命令一样。也可以用--decompress 或者--uncompress 选项来指定.
|-f	|强制压缩，即使原始文件的压缩文件已经存在了，也要执行。也可以用--force 选项来指定。
|-h	|显示用法信息。也可用--help 选项来指定。
|-l	|列出每个被压缩文件的压缩数据。也可用--list 选项。
|-r	|若命令的一个或多个参数是目录，则递归地压缩目录中的文件。也可用--recursive 选项来指定。
|-t	|测试压缩文件的完整性。也可用--test 选项来指定。
|-v	|显示压缩过程中的信息。也可用--verbose 选项来指定。
|-number	|设置压缩指数。number 是一个在1（最快，最小压缩）到9（最慢，最大压缩）之间的整数。 数值1和9也可以各自用--fast 和--best 选项来表示。默认值是整数6。


### bzip2

### tar

    tar mode[options] pathname...

部分 tar 模式

|模式|	说明|
|--|--|
|c	|为文件和／或目录列表创建归档文件。
|x	|抽取归档文件。
|r	|追加具体的路径到归档文件的末尾。
|t	|列出归档文件的内容。

详细参数：[tar参数列表](http://www.cnblogs.com/xxpal/articles/816691.html)

栗子：

	#与find命令结合并追加结果到归档文件
    [me@linuxbox ~]$ find playground -name 'file-A' -exec tar rf playground.tar '{}' '+' #这里我们使用 find 命令来匹配 playground 目录中所有名为 file-A 的文件，然后使用-exec 行为，来 唤醒带有追加模式（r）的 tar 命令，把匹配的文件添加到归档文件 playground.tar 里面。


	#tar 命令也可以利用标准输出和输入。这里是一个完整的例子:

	[me@linuxbox foo]$ cd
	[me@linuxbox ~]$ find playground -name 'file-A' | tar cf - --files-from=-
	   | gzip > playground.tgz
	   
>在这个例子里面，我们使用 find 程序产生了一个匹配文件列表，然后把它们管道到 tar 命令中。 如果指定了文件名“-”，则其被看作是标准输入或输出，正是所需（顺便说一下，使用“-”来表示 标准输入／输出的惯例，也被大量的其它程序使用）。这个 --file-from 选项（也可以用 -T 来指定） 导致 tar 命令从一个文件而不是命令行来读入它的路径名列表。最后，这个由 tar 命令产生的归档 文件被管道到 gzip 命令中，然后创建了压缩归档文件 playground.tgz。此 .tgz 扩展名是命名 由 gzip 压缩的 tar 文件的常规扩展名。有时候也会使用 .tar.gz 这个扩展名。

	#对归档文件进行gzip或bzip2压缩
    [me@linuxbox ~]$ find playground -name 'file-A' | tar czf playground.tgz -T -
    [me@linuxbox ~]$ find playground -name 'file-A' | tar cjf playground.tbz -T -

	#从远程复制
	[me@linuxbox ~]$ mkdir remote-stuff
	[me@linuxbox ~]$ cd remote-stuff
	[me@linuxbox remote-stuff]$ ssh remote-sys 'tar cf - Documents' | tar xf -
	me@remote-sys’s password:
	[me@linuxbox remote-stuff]$ ls
	Documents

>首先，通过使用 ssh 命令在远端系统中启动 tar 程序。你可记得 ssh 允许我们 在远程联网的计算机上执行程序，并且在本地系统中看到执行结果——远端系统中产生的输出结果 被发送到本地系统中查看。我们可以利用。在本地系统中，我们执行 tar 命令.

### zip

对于 zip 命令（与 tar 命令相反）要注意一点，就是如果指定了一个已经存在的文件包，其被更新 而不是被替代。这意味着会保留此文件包，但是会添加新文件，同时替换匹配的文件。可

    zip options zipfile file...

### rsync 

跳过

## 正则

### grep

“grep”这个名字 来自于短语“global regular expression print

    grep [options] regex [file...]

grep 选项

|选项|描述 |
|--|--|
|-i	|忽略大小写。不会区分大小写字符。也可用--ignore-case 来指定。|
|-v	|不匹配。通常，grep 程序会打印包含匹配项的文本行。这个选项导致 |grep |程序 只会不包含匹配项的文本行。也可用--invert-match 来指定。|
|-c	|打印匹配的数量（或者是不匹配的数目，若指定了-v 选项），而不是文本行本身。 也可用--count 选项来指定。|
|-l	|打印包含匹配项的文件名，而不是文本行本身，也可用--files-with-matches 选项来指定。|
|-L	|相似于-l 选项，但是只是打印不包含匹配项的文件名。也可用--files-without-match 来指定。|
|-n	|在每个匹配行之前打印出其位于文件中的相应行号。也可用--line-number 选项来指定。|
|-h	|应用于多文件搜索，不输出文件名。也可用--no-filename 选项来指定。|

正则表达式元字符：

    ^ $ . [ ] { } - ? * + ( ) | \


举个栗子：


    [me@linuxbox ~]$ ls dirlist*.txt
    dirlist-bin.txt     dirlist-sbin.txt    dirlist-usr-sbin.txt
    dirlist-usr-bin.txt
    
    [me@linuxbox ~]$ grep bzip dirlist*.txt 
    dirlist-bin.txt:bzip2
    dirlist-bin.txt:bzip2recover
    
    
    [me@linuxbox ~]$ grep -l bzip dirlist*.txt #只是对包含匹配项的文件列表
    dirlist-bin.txt
    
    
    [me@linuxbox ~]$ grep -L bzip dirlist*.txt #只想查看不包含匹配项的文件列表
    dirlist-sbin.txt
    dirlist-usr-bin.txt
    dirlist-usr-sbin.txt

    [me@linuxbox ~]$ grep -h '.zip' dirlist*.txt
    bunzip2
    bzip2
    bzip2recover
    gunzip

    [me@linuxbox ~]$ grep -i '^..j.r$' /usr/share/dict/words #在字典中查找单词
    Major
    major

### POSIX

POSIX 字符集

|字符集	| 说明  |
|--|--|
|[:alnum:]	|字母数字字符。在 ASCII 中，等价于：[A-Za-z0-9]|
|[:word:]	|与[:alnum:]相同, 但增加了下划线字符。|
|[:alpha:]	|字母字符。在 ASCII 中，等价于：[A-Za-z]|
|[:blank:]	|包含空格和 tab 字符。|
|[:cntrl:]	|ASCII 的控制码。包含了0到31，和127的 ASCII 字符。|
|[:digit:]	|数字0到9|
|[:graph:]	|可视字符。在 ASCII 中，它包含33到126的字符。|
|[:lower:]	|小写字母。|
|[:punct:]	|标点符号字符。在 ASCII 中，等价于：
|[:print:]	|可打印的字符。在[:graph:]中的所有字符，再加上空格字符。|
|[:space:]	|空白字符，包括空格，tab，回车，换行，vertical tab, 和 form feed.在 ASCII 中， 等价于：[ \t\r\n\v\f]|
|[:upper:]	|大写字母。|
|[:xdigit:]	|用来表示十六进制数字的字符。在 ASCII 中，等价于：[0-9A-Fa-f]|


    [me@linuxbox ~]$ ls /usr/sbin/[[:upper:]]*
    /usr/sbin/MAKEFLOPPIES
    /usr/sbin/NetworkManagerDispatcher
    /usr/sbin/NetworkManager

## 文本处理

### sed

	#对标准输入的内容，或命令行中指定的一个或多个文件进行排序，然后把排序 结果发送到标准输出。
    [me@linuxbox ~]$ sort > foo.txt
    c
    b
    a
    [me@linuxbox ~]$ cat foo.txt
    a
    b
    c

    sort file1.txt file2.txt file3.txt > final_sorted_list.txt

常见的 sort 程序选项

|选项|长选项	|描述|
|--|--|--|
|-b	|--ignore-leading-blanks	|默认情况下，对整行进行排序，从每行的第一个字符开始。这个选项导致 sort 程序忽略 每行开头的空格，从第一个非空白字符开始排序。|
|-f	|--ignore-case	|让排序不区分大小写。|
|-n	|--numeric-sort	|基于字符串的长度来排序。使用此选项允许根据数字值执行排序，而不是字母值。
-r	|--reverse	|按相反顺序排序。结果按照降序排列，而不是升序。
|-k	|--key=field1[,field2] 	|对从 field1到 field2之间的字符排序，而不是整个文本行。看下面的讨论。
|-m	|--merge	|把每个参数看作是一个预先排好序的文件。把多个文件合并成一个排好序的文件，而没有执行额外的排序。
|-o	|--output=file	|把排好序的输出结果发送到文件，而不是标准输出。
|-t	|--field-separator=char	|定义域分隔字符。默认情况下，域由空格或制表符分隔。

举例：

#du 命令可以 确定最大的磁盘空间用户。通常，这个 du 命令列出的输出结果按照路径名来排序：

    [me@linuxbox ~]$ du -s /usr/share/\* | head
    252     /usr/share/aclocal
    96      /usr/share/acpi-support
    8       /usr/share/adduser
    196     /usr/share/alacarte
    344     /usr/share/alsa
    8       /usr/share/alsa-base
    12488   /usr/share/anthy
    8       /usr/share/apmd
    21440   /usr/share/app-install
    48      /usr/share/application-registry

#把结果管道到 head 命令，把输出结果限制为前 10 行。我们能够产生一个按数值排序的 列表，来显示 10 个最大的空间消费者

    [me@linuxbox ~]$ du -s /usr/share/* | sort -nr | head
    509940         /usr/share/locale-langpack
    242660         /usr/share/doc
    197560         /usr/share/fonts
    179144         /usr/share/gnome
    146764         /usr/share/myspell
    144304         /usr/share/gimp
    135880         /usr/share/dict
    76508          /usr/share/icons
    68072          /usr/share/apps
    62844          /usr/share/foomatic

	#忽略 ls 程序能按照文件大小对输出结果进行排序，我们也能够使用 sort 程序来完成此任务
    [me@linuxbox ~]$ ls -l /usr/bin | sort -nr -k 5 | head
    -rwxr-xr-x 1 root   root   8234216  2008-04-0717:42 inkscape
    -rwxr-xr-x 1 root   root   8222692  2008-04-07 17:42 inkview


    #我们指定了 1,1， 意味着“始于并且结束于第一个字段。”在第二个实例中，我们指定了 2n，意味着第二个字段是排序的键值， 并且按照数值排序。
    [me@linuxbox ~]$ sort --key=1,1 --key=2n distros.txt
    Fedora         5     03/20/2006
    Fedora         6     10/24/2006
    Fedora         7     05/31/2007
    
    #通过指定 -k 3.7，我们指示 sort 程序使用一个排序键值，其始于第三个字段中的第七个字符，对应于 年的开头。同样地，我们指定 -k 3.1和 -k 3.4来分离日期中的月和日。 我们也添加了 n 和 r 选项来实现一个逆向的数值排序。
    [me@linuxbox ~]$ sort -k 3.7nbr -k 3.1nbr -k 3.4nbr distros.txt
    Fedora         10    11/25/2008
    Ubuntu         8.10  10/30/2008
    SUSE           11.0  06/19/2008
    
    #一些文件不会使用 tabs 和空格做为字段界定符
    #按照第七个字段（帐户的默认 shell）来排序此 passwd 文件
    [me@linuxbox ~]$ sort -t ':' -k 7 /etc/passwd | head
    me:x:1001:1001:Myself,,,:/home/me:/bin/bash
    root:x:0:0:root:/root:/bin/bash
    dhcp:x:101:102::/nonexistent:/bin/false
    gdm:x:106:114:Gnome Display Manager:/var/lib/gdm:/bin/false
    hplip:x:104:7:HPLIP system user,,,:/var/run/hplip:/bin/false
    klog:x:103:104::/home/klog:/bin/false
    messagebus:x:108:119::/var/run/dbus:/bin/false
    polkituser:x:110:122:PolicyKit,,,:/var/run/PolicyKit:/bin/false
    pulse:x:107:116:PulseAudio daemon,,,:/var/run/pulse:/bin/false

### uniq

uniq可进行排序，其输入必须是排好序的数据，因为 uniq 只会删除相邻的重复行。

    [me@linuxbox ~]$ sort foo.txt | uniq
    a
    b
    c

	#报告文本文件中重复行的次数，使用这个-c 选项：
	[me@linuxbox ~]$ sort foo.txt | uniq -c
	        2 a
	        2 b
	        2 c

常用的 uniq 选项

	
| 选项 |说明  |
|--|--|
|-c	|输出所有的重复行，并且每行开头显示重复的次数。
|-d	|只输出重复行，而不是特有的文本行。
|-f n	|忽略每行开头的 n 个字段，字段之间由空格分隔，正如 sort 程序中的空格分隔符；然而， 不同于 sort 程序，uniq 没有选项来设置备用的字段分隔符。
|-i	|在比较文本行的时候忽略大小写。
|-s n	|跳过（忽略）每行开头的 n 个字符。
|-u	|只是输出独有的文本行。这是默认的。

### 切片和切块

#### cut

作用：从文本行中抽取文本，并把其输出到标准输出。它能够接受多个文件参数或者 标准输入。


    [me@linuxbox ~]$ cat -A distros.txt
    SUSE^I10.2^I12/07/2006$
    Fedora^I10^I11/25/2008$
    SUSE^I11.0^I06/19/2008$
    Ubuntu^I8.04^I04/24/2008$  
    [me@linuxbox ~]$ cut -f 3 distros.txt
    12/07/2006
    11/25/2008
    06/19/2008
    04/24/2008
    [me@linuxbox ~]$ cut -f 3 distros.txt | cut -c 7-10
	2006
	2008
	2007
	2006

	#从/etc/passwd 文件中 抽取第一个字段：
	#使用-d 选项，我们能够指定冒号做为字段分隔符。
	[me@linuxbox ~]$ cut -d ':' -f 1 /etc/passwd | head
	root
	daemon
	bin
	sys
	sync
	games
	man
	lp
	mail
	news

cut 程序选择项

|选项	|说明    |
|--|--|
|-c char_list	|从文本行中抽取由 char_list 定义的文本。这个列表可能由一个或多个逗号 分隔开的数值区间组成。
|-f field_list	|从文本行中抽取一个或多个由 field_list 定义的字段。这个列表可能 包括一个或多个字段，或由逗号分隔开的字段区间。
|-d delim_char	|当指定-f 选项之后，使用 delim_char 做为字段分隔符。默认情况下， 字段之间必须由单个 tab 字符分隔开。
|--complement	|抽取整个文本行，除了那些由-c 和／或-f 选项指定的文本。

#### paste

#### join

### 比较文本
	[me@linuxbox ~]$ cat > file1.txt
	a
	b
	c
	d
	[me@linuxbox ~]$ cat > file2.txt
	b
	c
	d
	e
	[me@linuxbox ~]$ comm file1.txt file2.txt
	a
	        b
	        c
	        d
	    e

	#comm 命令产生了三列输出。第一列包含第一个文件独有的文本行；第二列， 文本行是第二列独有的；第三列包含两个文件共有的文本行。comm 支持 -n 形式的选项，这里 n 代表 1，2 或 3。这些选项使用的时候，指定了要隐藏的列。
	
	#只想输出两个文件共享的文本行， 我们将隐藏第一列和第二列的输出结果：
	[me@linuxbox ~]$ comm -12 file1.txt file2.txt
	b
	c
	d
#### diff
#### patch

### 运行时编辑

#### tr

	[me@linuxbox ~]$ echo "lowercase letters" | tr a-z A-Z
	LOWERCASE LETTERS

	[me@linuxbox ~]$ echo "lowercase letters" | tr [:lower:] A
	AAAAAAAAA AAAAAAA


	#挤压
	[me@linuxbox ~]$ echo "aaabbbccc" | tr -s ab
	abccc

### sed


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
|Red Hat Style (.rpm)	|Fedora, CentOS, Red Hat Enterprise Linux, OpenSUSE, Mandriva, PCLinuxOS

### 包文件

在包管理系统中软件的基本单元是包文件。包文件是一个构成软件包的文件压缩集合。

### 上层和底层软件包工具

软件包管理系统通常由两种工具类型组成：底层工具用来处理这些任务，比方说安装和删除软件包文件， 和上层工具，完成元数据搜索和依赖解析。

||||
|---|---|---|
|发行版	|底层工具|上层工具|
|Debian-Style	|dpkg|	apt-get, aptitude
|Fedora, Red Hat Enterprise Linux, CentOS|	rpm	|yum





