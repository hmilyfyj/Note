title: 利用github的webhooks自动部署hexo到github pages
date: 2015-12-3 22:27:3
tags: [折腾,hexo,git,webhooks,vps,php]
categories: 折腾
---

未完成

## 方案1

### Git 设置 webhooks

### PHP 创建更新标志文件

    /**
	 * 配合GIT自动部署
	 *
	 * @author Feng <fengit@shanjing-inc.com>
	 */
	public function deploy() {
	    echo file_put_contents(APPPATH.'cache/git_update.txt', 'I made some changes.');
	}


### VPS

#### 创建脚本，检测文件是否发生改动

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
