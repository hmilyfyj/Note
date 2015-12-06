<p>title: PHP知识点<br>
date: 2015-12-3<br>
tags: [服务端IP,]<br>
categories: 折腾</p>
<hr>
<h3 id="踩过的坑">踩过的坑</h3>
<h4 id="using-this-when-not-in-object-context-in"><code>Using $this when not in object context in</code></h4>
<p>静态方法内无法使用$this,解决方法：</p>
<pre><code>self::method();
</code></pre>
<p><a href="http://blog.csdn.net/yageeart/article/details/6662059">参考地址</a></p>
<h3 id="小tips">小TIPS</h3>
<h4 id="获取物理路径">获取物理路径</h4>
<pre><code>realpath();
</code></pre>