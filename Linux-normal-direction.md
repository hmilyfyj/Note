title: Linux命令总结
date: 2016-03-08 21:29
tags: [折腾,Linux]
categories: Linux
---

系统的过一遍命令，然后根据需求专精。

[The Linux Command Line](http://www.kancloud.cn/thinkphp/linux-command-line/39431)

<!-- more -->

---
# 命令
## 文件系统
- pwd — 打印出当前工作目录名
- cd — 更改目录
- ls — 列出目录内容
- ls — 列出目录内容
- file — 确定文件类型
- less — 浏览文件内容

### 文件操作
- cp — 复制文件和目录
- mv — 移动/重命名文件和目录
- mkdir — 创建目录
- rm — 删除文件和目录
- ln — 创建硬链接和符号链接


#### 关于软硬连接，这里举个栗子：

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

##### 总结
1. 删除符号连接f3,对f1,f2无影响；
2. 删除硬连接f2，对f1,f3也无影响；
3. 删除原文件f1，对硬连接f2没有影响，导致符号连接f3失效；
4. 同时删除原文件f1,硬连接f2，整个文件会真正的被删除。

#### 查找
- locate – 通过名字来查找文件
- find – 在目录层次结构中搜索文件
我们也将看一个经常与文件搜索命令一起使用的命令，它用来处理搜索到的文件列表：
- xargs – 从标准输入生成和执行命令行
另外，我们将介绍两个命令来协助我们探索：
-  touch – 更改文件时间
- stat – 显示文件或文件系统状态


#### 归档和备份


- gzip – 压缩或者展开文件
- bzip2 – 块排序文件压缩器


归档程序：


- tar – 磁带打包工具
- zip – 打包和压缩文件


还有文件同步程序：


- rsync – 同步远端文件和目录


## 重定向
- cat － 连接文件
- sort － 排序文本行
- uniq － 报道或省略重复行
- grep － 打印匹配行
- wc － 打印文件中换行符，字，和字节个数
- head － 输出文件第一部分
- tail - 输出文件最后一部分

## 屏幕显示
- clear － 清空屏幕
- history － 显示历史列表内容
## 权限
- id – 显示用户身份号
- chmod – 更改文件模式
- umask – 设置默认的文件权限
- su – 以另一个用户的身份来运行 shell
- sudo – 以另一个用户的身份来执行命令
- chown – 更改文件所有者
- chgrp – 更改文件组所有权
- passwd – 更改用户密码

## 进程管理
- ps – 报告当前进程快照
- top – 显示任务
- jobs – 列出活跃的任务
- bg – 把一个任务放到后台执行
- fg – 把一个任务放到前台执行
- kill – 给一个进程发送信号
- killall – 杀死指定名字的进程
- shutdown – 关机或重启系统

## shell环境
- printenv - 打印部分或所有的环境变量
- set - 设置 shell 选项
- export — 导出环境变量，让随后执行的程序知道。
- alias - 创建命令别名

## 存储媒介
- mount – 挂载一个文件系统
- umount – 卸载一个文件系统
- fsck – 检查和修复一个文件系统
- fdisk – 分区表控制器
- mkfs – 创建文件系统
- fdformat – 格式化一张软盘
- dd — 把面向块的数据直接写入设备
- genisoimage (mkisofs) – 创建一个 ISO 9660的映像文件
- wodim (cdrecord) – 把数据写入光存储媒介
- md5sum – 计算 MD5检验码

## 网络

- ping - 发送 ICMP ECHO_REQUEST 软件包到网络主机
- traceroute - 打印到一台网络主机的路由数据包
- netstat - 打印网络连接，路由表，接口统计数据，伪装连接，和多路广播成员
- ftp - 因特网文件传输程序
- wget - 非交互式网络下载器
- ssh - OpenSSH SSH 客户端（远程登录程序）

## 文本处理
- cat – 连接文件并且打印到标准输出
- sort – 给文本行排序
- uniq – 报告或者省略重复行
- cut – 从每行中删除文本区域
- paste – 合并文件文本行
- join – 基于某个共享字段来联合两个文件的文本行
- comm – 逐行比较两个有序的文件
- diff – 逐行比较文件
- patch – 给原始文件打补丁
- tr – 翻译或删除字符
- sed – 用于筛选和转换文本的流编辑器
- aspell – 交互式拼写检查器
 
## 格式化输出
- nl – 添加行号
- fold – 限制文件列宽
- fmt – 一个简单的文本格式转换器
- pr – 让文本为打印做好准备
- printf – 格式化数据并打印出来
- groff – 一个文件格式系统

## 打印
- pr —— 转换需要打印的文本文件
- lpr —— 打印文件
- lp —— 打印文件（System V）
- a2ps —— 为 PostScript 打印机格式化文件
- lpstat —— 显示打印机状态信息
- lpq —— 显示打印机队列状态
- lprm —— 取消打印任务
- cancel —— 取消打印任务（System V）

## 编译程序
- ./configure
- make
- make install

# 快捷键

## 光标移动命令
| 按键 |行动|
|--|--|
| Ctrl-a |移动光标到行首。  |
| Ctrl-e |移动光标到行尾。  |
| Ctrl-f |光标前移一个字符；和右箭头作用一样。  |
| Ctrl-b |光标后移一个字符；和左箭头作用一样。  |
| Alt-a | 光标前移一个字。 |
| Alt-b |光标后移一个字。  |
| Ctrl-l |清空屏幕，移动光标到左上角。clear 命令完成同样的工作。 |

## 修改文本

|按键	|行动|
|--|--|
|Ctrl-d|	|删除光标位置的字符。|
|Ctrl-t|	|光标位置的字符和光标前面的字符互换位置。|
|Alt-t	|光标位置的字和其前面的字互换位置。|
|Alt-l	|把从光标位置到字尾的字符转换成小写字母。|
|Alt-u	|把从光标位置到字尾的字符转换成大写字母。|

## 剪切和粘贴文本
|按键	|行动|
|--|--|
|Ctrl-k|	剪切从光标位置到行尾的文本。|
|Ctrl-u|	剪切从光标位置到行首的文本。|
|Alt-d|	剪切从光标位置到词尾的文本。|
|Alt-Backspace|	剪切从光标位置到词头的文本。如果光标在一个单词的开头，剪切前一个单词。|
|Ctrl-y|	把剪切环中的文本粘贴到光标位置。|

## 
