title: Centos 下安装 expect
date: 2016-04-25 18:41
tags: [centos,expect]
categories: Centos
---

<!-- more -->
---

# 安装 Tcl

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
    
# 安装 expect

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


