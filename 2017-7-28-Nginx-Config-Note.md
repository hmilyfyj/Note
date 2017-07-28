---
title: 2017-7-28-Nginx-Config-Note
date:  2017-07-28 9:0:30
tags: Note
categories: Note
grammar_cjkRuby: true
---

本篇笔记记录 `Nginx` 相关的内容。

<!-- more -->

---

``` nginx
location  = / {
  # 精确匹配 / ，主机名后面不能带任何字符串
  [ configuration A ]
}
location  / {
  # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
  # 但是正则和最长字符串会优先匹配
  [ configuration B ]
}
location /documents/ {
  # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration C ]
}
location ~ /documents/Abc {
  # 匹配任何以 /documents/Abc 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration CC ]
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
  [ configuration D ]
}
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配所有以 gif,jpg或jpeg 结尾的请求
  # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
  [ configuration E ]
}
location /images/ {
  # 字符匹配到 /images/，继续往下，会发现 ^~ 存在
  [ configuration F ]
}
location /images/abc {
  # 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在
  # F与G的放置顺序是没有关系的
  [ configuration G ]
}
location ~ /images/abc/ {
  # 只有去掉 config D 才有效：先最长匹配 config G 开头的地址，继续往下搜索，匹配到这一条正则，采用
    [ configuration H ]
}
location ~* /js/.*/\.js
```

只允许

``` stylus
	location ~* (\.txt|\.html|\.htm)$ {
		location ~ robots\.txt$ {
		allow all;
	}
	
	error_page 403 =404 /index.php;
	deny all;
}
```

http://seanlook.com/2015/05/17/nginx-location-rewrite/
https://blog.skipfault.com/2013/05/09/nginx-block-all-text-files-except-robots-txt/


