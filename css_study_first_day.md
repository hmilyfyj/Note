title: CSS 学习笔记（一） ：选择器、
date: 2015-12-24 22:29
tags: [css,学习]
categories: CSS
---

- 行内元素：
- 替换元素：
- 非替换元素

 [介绍链接](http://blog.csdn.net/chenmoquan/article/details/44646369)

### 选择：

    body,table {color:gray} //分组选择
	.warning.uragent {color:gray} //类选择
	plate[moon]  {color:gray} //包含moon属性的 palte
	h1 em  {color:gray} //继承
	
	h1[class*="test"] class 包含 test 的 h1
	h1[class~="test"] class 包含 test 的 h1
	h1[class$="test"] 以 test 结尾
	h1[class^="test"] 以 test 开头
	
	p>strong 选择子元素 strong //父子
	p+h 选择 p 后面的 h 元素 //兄弟
	a:visted //伪类
	
	:focus 焦点 :hover 鼠标停留：active 点击后激活
	
	li:first-child //选择作为第一个子元素的li元素

[伪元素、伪类的区别](http://segmentfault.com/a/1190000000484493)

Mark : Page 63.

    h2:before {content:"abc";color:silver;} //h2之前添加银色的 }}
	
###  结构与层叠
specificity计算方法：

 

    h1 {color:read !important;} //重要性，加在分号前。
    LVHA

[读者、创作、代理样式
](http://blog.sina.com.cn/s/blog_4398c8d30100zht5.html)

### 值、单位
#### 关键字：
数字、百分比、颜色（17个）
安全色
绝对长度：1in = 2.54cm = 25.4mm = 72pt = 6pc 

> PAGE:92

相对长度：
- em：相对于字体的度量
- px：屏幕上的一个点，取决于分辨率

引入资源：
- css文件中引入：@import url(special/gif.gif);
- html中 `<link rel="stylesheet" type ="text/css" href="path">`

关键字

- inherit 继承父元素的值

### 字体

- font-family:'New York',Serify;
- font-weight: //粗细
- font-size: //大小
> PAGE 110

缩进：
- text-indent:缩进
- text-align:left|center|right|justify
- line-height:
![enter image description here](http://ww4.sinaimg.cn/large/c048f998gw1ezczrob8hyj20ob05yt9t.jpg)

对齐：
vertical-align: //应用于行内元素、替换元素
word-spacing://单词间隔
letter-spacing://字母间隔
text-tranform:大小写转换

