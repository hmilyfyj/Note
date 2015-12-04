
title: 利用github的webhooks自动部署hexo到github pages
date: 2015-12-3 22:27:3
tags: [折腾,hexo,git,webhooks,vps,php]
categories: 折腾
---




> 前言：Hexo没有后台编辑，如果想发表文章最基本的步骤是 `hexo new`创建文章后编辑新建的`xx.md` 文件，然后`hexo d -g` 部署。这在我看来一是不能随身携带，换了电脑再部署很麻烦，而且错做繁琐，为什么不把这些重复的工作交给电脑，我只要开开心心编写笔记呢？于是有了如下的折腾。

----------


## 方案1

### Git


> 创建Note项目存放markdown笔记，并且设置webhooks

![](http://7xnocp.com1.z0.glb.clouddn.com/15-12-4/19076981.jpg)

<!--more-->

### PHP

> 实现上一步的回调接口，功能：向指定文件夹写入文件

    /**
	 * 配合GIT自动部署
	 *
	 * @author Feng <fengit@shanjing-inc.com>
	 */
	public function deploy() {
	    echo file_put_contents(APPPATH.'cache/git_update.txt', 'I made some changes.');
	}


### VPS

#### 创建脚本，检测上一步骤中的创建的文件是否存在，存在则进行更新、部署。

    #!/bin/sh
    update_flag="/var/www/www.fengbl.cn/application/cache/git_update.txt"
    hexo_dir="/root/blog"
    post_dir="/root/blog/source/_posts/"
    log_dir="/var/www/www.fengbl.cn"
    
    cur_time=`date "+%Y-%m-%d %H:%M:%S"`
    
    if [ -f "$update_flag" ];then
    	cd "$log_dir"
    	echo "$cur_time：Changed." >> update_hexo_flag.txt 2>&1
    	echo >> update_hexo_flag.txt
    	rm "$update_flag"
    	cd "$post_dir"
    	git pull origin master
    	cd "$hexo_dir"
    	export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/dist
    	source ~/git/nvm/nvm.sh
    	nvm use v5.1.0
    	hexo d -g
    	hexo b
    fi
    
    
### 方案2

省去第二部，直接由PHP执行shell脚本，由于不够安全，放弃。

