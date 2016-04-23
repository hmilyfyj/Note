title: 人生苦短，学学 Docker
date: 2016-04-22 18:38
tags: [折腾,Docker]
categories: Docker
---

饱尝部署环境之麻烦，决定花点时间学 Docker，看看能不能偷懒 & 一劳永逸。

<!-- more -->

---

# 常用命令

交互命令：

    docker run -i -t  daocloud.io/hmilyfyj/memcached:master-a9a55bf  /bin/bash
    
大扫除（清除所有容器 && 镜像）：

      docker kill $(docker ps -q) && docker rm $(docker ps -a -q)
      docker rmi $(docker images -q -a) 


