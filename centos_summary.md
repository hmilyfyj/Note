title: Centos的一些总结
date: 2015-12-3 22:19:46
tags: [折腾,centos,sftp]
categories: Centos
---

### 开启 sftp

#### 创建sftp组

    groupadd sftp

#### 创建一个sftp用户，名为mysftp，密码为mysftp

    useradd -g sftp -s /bin/false mysftp
    passwd mysftp 


#### sftp组的用户的home目录统一指定到/data/sftp下，按用户名区分，这里先新建一个mysftp目录，然后指定mysftp的home为/data/sftp/mysftp

    mkdir -p /data/sftp/mysftp  
    usermod -d /data/sftp/mysftp mysftp  
    
#### 配置sshd_config

    vi /etc/ssh/sshd_config

找到如下这行，用#符号注释掉，大致在文件末尾处。
    # Subsystem      sftp    /usr/libexec/openssh/sftp-server  
在文件最后面添加如下几行内容

    Subsystem       sftp    internal-sftp    
    Match Group sftp    
    ChrootDirectory /data/sftp/%u    
    ForceCommand    internal-sftp    
    AllowTcpForwarding no    
    X11Forwarding no  


#### 设定Chroot目录权限

    chown root:sftp /data/sftp/mysftp  
    chmod 755 /data/sftp/mysftp

#### 建立SFTP用户登入后可写入的目录

> 照上面设置后，在重启sshd服务后，用户mysftp已经可以登录。但使用chroot指定根目录后，根应该是无法写入的，所以要新建一个目录供mysftp上传文件。这个目录所有者为mysftp，所有组为sftp，所有者有写入权限，而所有组无写入权限。命令如下：

    mkdir /data/sftp/mysftp/upload  
    chown mysftp:sftp /data/sftp/mysftp/upload  
    chmod 755 /data/sftp/mysftp/upload 

#### 修改 /etc/selinux/config

    vi /etc/selinux/config  

将文件中的SELINUX=enforcing 修改为 SELINUX=disabled

再输入命令：

    setenforce 0  
    
#### 重启sshd服务

service sshd restart  
