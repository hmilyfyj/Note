title: Hexo 采坑集锦
date: 2016-02-13 10:24
tags: [折腾,hexo]
categories: 
---

# 错误1

    ERROR Plugin load failed: hexo-renderer-marked

## 解决办法

    生成的 `package.json`  
    "hexo-renderer-marked": "^0.1.0",  版本有问题。解决：
    
    sudo npm install hexo-renderer-marked@0.2.3

 

