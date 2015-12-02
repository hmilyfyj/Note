title: 使用 Hexo
date: 2014-11-21 12:03:30
tags: [Hexo]
categories: 折腾
---

### 一、安装 node.js

#### install nvm
    $ cd ~/git
    $ git clone https://github.com/creationix/nvm.git

配置终端启动时自动执行 `source ~/git/nvm/nvm.sh`
在 `~/.bashrc`, `~/.bash_profile`, `~/.profile`, 或者 `~/.zshrc` 文件添加以下命令后重新打开终端:

    source ~/git/nvm/nvm.sh

#### install node.js

    nvm install 0.11.11
##### 国内加速安装：

    NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/dist nvm install 0.11.11

为了不每次都输入，可以修改`.bashrc`文件，加入如下内容：

    # nvm
    export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/dist
    source ~/git/nvm/nvm.sh

### 二、安装 hexo 

    $ npm install hexo-cli -g
    $ hexo init blog
	$ cd blog
	$ npm install
[Hexo-Github](https://github.com/hexojs/hexo)

#### install plugins

##### hexo-back-up

    npm install hexo-git-backup --save

[Hexo-back-up Github](https://github.com/coneycode/hexo-git-backup)


##### 国内加速安装：

	npm --registry=https://registry.npm.taobao.org install cnpm -g

以后就可以用 cnpm 替代 npm 了。


