
title: Markdown中增加首行缩进
date: 2016-02-22 12:09
tags: [Hexo,日常,在成为最厉害最厉害最厉害的路上]
categories: Hexo

---

中文段落不加首行缩进很难看，总结了两种方法。

<!-- more -->

## 方法1


半方大的空白`&ensp;`或`&#8194;`
全方大的空白`&emsp;`或`&#8195;`
不断行的空白格`&nbsp;`或`&#160;`


## 方法2

修改 `/source/css/style.stl`

添加

    p {text-indent 2em;｝

相关文章：

http://www.jianshu.com/p/9d94660a96f1

http://coolrc.top/2015/11/29/hexo-first-line-indent/


