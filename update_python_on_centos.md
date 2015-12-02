title: Centos 下 Python2.6 => 2.7
date: 2015-11-29 11:7:57
tags: [Centos]
categories: Centos
---

### 升级Python

#### 下载

    cd /tmp
    wget https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tar.xz

[更多 Python 版本][1]

#### 解压
    tar -Jcf Python-2.7.10.tar.xz

[更多解压方式][2]

<!--more-->

#### 安装
    ./configure
    make all
    make install
    make clean
    make distclean

#### 查看刚刚安装得Python版本：
    /usr/local/bin/python2.7 -V


#### 将系统默认的python指向到2.7版本。

    mv /usr/bin/python /usr/bin/python2.6.6
    ln -s /usr/local/bin/python2.7 /usr/bin/python

### 常见问题

#### 修复 yum
    vim /usr/bin/yum
    #!/usr/bin/python => #!/usr/bin/python2.6.6

  [1]: https://www.python.org/downloads/release/python-2710/
  [2]: https://teddysun.com/294.html
