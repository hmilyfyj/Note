title: Expect -- Shell 交互工具
date: 2016-04-25 18:41
tags: [Centos,Expect]
categories: Centos
---

<!-- more -->
---


# 安装

## 一、Tcl

主页: http://www.tcl.tk

    wget http://120.52.73.45/jaist.dl.sourceforge.net/project/tcl/Tcl/8.4.20/tcl8.4.20-src.tar.gz

解压：
    
    tar xfvz tcl8.4.20-src.tar.gz
    
    cd tcl8.4.20/unix

编译、安装
    
    ./configure --prefix=/usr/local/tcl --enable-shared
    make
    make install
    
安装完毕以后，进入tcl源代码的根目录，把子目录unix下面的tclUnixPort.h copy到子目录generic中。

    cd /tmp && cp tcl8.4.20/unix/tclUnixPort.h tcl8.4.20/generic/
    
组合成一条命令：

```
cd /tmp && wget http://120.52.73.45/jaist.dl.sourceforge.net/project/tcl/Tcl/8.4.20/tcl8.4.20-src.tar.gz  && tar xfvz tcl8.4.20-src.tar.gz && cd tcl8.4.20/unix && ./configure --prefix=/usr/local/tcl --enable-shared && make && make install

cd /tmp && cp tcl8.4.20/unix/tclUnixPort.h tcl8.4.20/generic/
```
    
## 二、Expect

下载

    wget http://120.52.73.45/nchc.dl.sourceforge.net/project/expect/Expect/5.45/expect5.45.tar.gz
    
解压

    tar xzvf expect5.45.tar.gz
    
    cd expect5.45

编译、安装

    ./configure --prefix=/usr/local/expect --with-tcl=/usr/local/tcl/lib --with-tclinclude=../tcl8.4.20/generic 
    
    make
    
    make install
    
做下连接

    ln -s /usr/local/tcl/bin/expect /usr/bin/expect
    
组合成一条命令：

```
cd /tmp && wget http://120.52.73.45/nchc.dl.sourceforge.net/project/expect/Expect/5.45/expect5.45.tar.gz && tar xzvf expect5.45.tar.gz && cd expect5.45 && ./configure --prefix=/usr/local/expect --with-tcl=/usr/local/tcl/lib --with-tclinclude=../tcl8.4.20/generic && make && make install && ln -s /usr/local/tcl/bin/expect /usr/local/expect/bin/expect
```

# 使用方法

简单使用：

```
#!/usr/bin/expect 

 

 # 设置超时时间为 60 秒

 set timeout  60                                         

 # 设置要登录的主机 IP 地址

 set host 192.168.1.46

 # 设置以什么名字的用户登录

 set name root 

 # 设置用户名的登录密码

 set password 123456 

 

 #spawn 一个 ssh 登录进程

 spawn  ssh $host -l $name 

 # 等待响应，第一次登录往往会提示是否永久保存 RSA 到本机的 know hosts 列表中；等到回答后，在提示输出密码；之后就直接提示输入密码

 expect { 

    "(yes/no)?" { 

        send "yes\n"

        expect "assword:"

        send "$pasword\n"

    } 

        "assword:" { 

        send "$password\n"

    } 

 } 

 expect "#"

 # 下面测试是否登录到 $host 

 send "uname\n"

 expect "Linux"

 send_user  "Now you can do some operation on this terminal\n"

 # 这里使用了 interact 命令，使执行完程序后，用户可以在 $host 终端进行交互操作。

 Interact 
```


传参：

    set password [lrange $argv 0 0]
    
切换工作目录：

    cd PATH
    

[参考资料][1]
[参考资料][2]


  [1]: http://www.cnblogs.com/iloveyoucc/archive/2012/05/11/2496433.html
  [2]: http://blog.csdn.net/zhuying_linux/article/details/6904568