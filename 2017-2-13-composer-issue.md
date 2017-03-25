title: Composer 踩坑集锦
date: 2017-02-13 09:12:22
tags: [Composer,issue]
categories: Composer
---

test

<!-- more -->

---



## 无法安装

执行：
````shell
composer require ltsteam/oapi-taobao-client
````

报错：

````shell
  [InvalidArgumentException]
  Could not find package ltsteam/oapi-taobao-client at any version for your minimum-stability (stable). Check the pac
  kage spelling or your minimum-stability
````

解决：

````shell
composer require orzcc/taobao-top-client dev-master
````