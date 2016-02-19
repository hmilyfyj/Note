
title: CSRF笔记
date: 2016-02-19 16:42
tags: [白帽子]
categories: 白帽子

---

这几天在阅读CI源码时同样看到了针对CSRF相关的安全措施。以前用扫描器会提示CSRF漏洞，当时不晓得是什么意思也没有深究，现在补上。

<!-- more -->


# CSRF是什么？

## 定义

CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

简单讲，就是“伪造请求，冒充用户在站内的正常操作。”

![](http://7xnocp.com1.z0.glb.clouddn.com/16-2-19/74578369.jpg)

## 举个栗子

### EG1:

**银行网站A**，它以GET请求来完成银行转账的操作，如：http://www.mybank.com/Transfer.php?toBankId=11&money=1000

**危险网站B**，它里面有一段HTML的代码如下：

    　<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>

首先，你登录了银行网站A，然后访问危险网站B，噢，这时你会发现你的银行账户少了1000块......

#### 原因

> 银行网站A违反了HTTP规范，使用GET请求更新资源。在访问危险网站B的之前，你已经登录了银行网站A，而B中的img 以GET的方式请求第三方资源（这里的第三方就是指银行网站了，原本这是一个合法的请求，但这里被不法分子利用了），所以你的浏览器会带上你的银行网站A的Cookie发出Get请求，去获取资源“http://www.mybank.com/Transfer.php?toBankId=11&money=1000”，结果银行网站服务器收到请求后，认为这是一个更新资源操作（转账操作），所以就立刻进行转账操作......

### EG2:

为了杜绝上面的问题，银行决定改用POST请求完成转账操作。

银行网站A的WEB表单如下：　　

```html
　　<form action="Transfer.php" method="POST">
　　　　<p>ToBankId: <input type="text" name="toBankId" /></p>
　　　　<p>Money: <input type="text" name="money" /></p>
　　　　<p><input type="submit" value="Transfer" /></p>
　　</form>
```
后台处理页面Transfer.php如下：
　　
```php
　　<?php
　　　　session_start();
　　　　if (isset($_REQUEST['toBankId'] &&　isset($_REQUEST['money']))
　　　　{
　　　　    buy_stocks($_REQUEST['toBankId'],　$_REQUEST['money']);
　　　　}
　　?>
```

危险网站B，仍然只是包含那句HTML代码：

```html
<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>
```


> 和示例1中的操作一样，你首先登录了银行网站A，然后访问危险网站B，结果.....和示例1一样，你再次没了1000块～T_T，这次事故的原因是：银行后台使用了`$_REQUEST`去获取请求的数据，而`$_REQUEST`既可以获取GET请求的数据，也可以获取POST请求的数据，这就造成了在后台处理程序无法区分这到底是GET请求的数据还是POST请求的数据。在PHP中，可以使用`$_GET`和`$_POST`分别获取GET请求和POST请求的数据。在JAVA中，用于获取请求数据request一样存在不能区分GET请求数据和POST数据的问题。

##### 关于$_REQUEST

> 由于 $_REQUEST 中的变量通过 GET，POST 和 COOKIE 输入机制传递给脚本文件，因此可以被远程用户篡改而并不可信。这个数组的项目及其顺序依赖于 PHP 的 variables_order  指令的配置。

**扩展知识点：register_globals**

　　
### EG3:

经过前面2个惨痛的教训，银行决定把获取请求数据的方法也改了，改用$_POST，只获取POST请求的数据，后台处理页面Transfer.php代码如下：

```php
　<?php
　　　　session_start();
　　　　if (isset($_POST['toBankId'] &&　isset($_POST['money']))
　　　　{
　　　　    buy_stocks($_POST['toBankId'],　$_POST['money']);
　　　　}
　　?>
```

然而。。。


```html
<html>
　　<head>
　　　　<script type="text/javascript">
　　　　　　function steal()
　　　　　　{
          　　　　 iframe = document.frames["steal"];
　　     　　      iframe.document.Submit("transfer");
　　　　　　}
　　　　</script>
　　</head>

　　<body onload="steal()">
　　　　<iframe name="steal" display="none">
　　　　　　<form method="POST" name="transfer"　action="http://www.myBank.com/Transfer.php">
　　　　　　　　<input type="hidden" name="toBankId" value="11">
　　　　　　　　<input type="hidden" name="money" value="1000">
　　　　　　</form>
　　　　</iframe>
　　</body>
</html>
```


　　总结一下上面3个例子，CSRF主要的攻击模式基本上是以上的3种，其中以第1,2种最为严重，因为触发条件很简单，一个<img>就可以了，而第3种比较麻烦，需要使用JavaScript，所以使用的机会会比前面的少很多，但无论是哪种情况，只要触发了CSRF攻击，后果都有可能很严重。

　　理解上面的3种攻击模式，其实可以看出，CSRF攻击是源于WEB的隐式身份验证机制！WEB的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却**无法保证该请求是用户批准发送的**！

### 实际案例

[WooYun: TP-LINK路由器CSRF，可干许多事（影响使用默认密码或简单密码用户）](http://www.wooyun.org/bugs/wooyun-2013-026825)

[案例2](http://wooyun.org/bugs/wooyun-2010-026622)

[案例3](http://wooyun.org/bugs/wooyun-2010-022895)

## 应对CSRF


1. 关键操作只接受POST请求

2. 验证码

> CSRF攻击的过程，往往是在用户不知情的情况下构造网络请求。所以如果使用验证码，那么每次操作都需要用户进行互动，从而简单有效的防御了CSRF攻击。

不足：

> 严重影响用户体验，所以验证码一般只出现在特殊操作里面，或者在注册时候使用

3. 检测refer

> 常见的互联网页面与页面之间是存在联系的，比如你在www.baidu.com应该是找不到通往www.google.com的链接的，再比如你在论坛留言，那么不管你留言后重定向到哪里去了，之前的那个网址一定会包含留言的输入框，这个之前的网址就会保留在新页面头文件的Referer中

不足：

> 问题出在服务器不是任何时候都能接受到Referer的值，所以Refere Check 一般用于监控CSRF攻击的发生，而不用来抵御攻击。

4. Cookie Hashing

因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败了:>


```php
　　<?php
　　　　//构造加密的Cookie信息
　　　　$value = “DefenseSCRF”;
　　　　setcookie(”cookie”, $value, time()+3600);
　　?>
```
在表单里增加Hash值，以认证这确实是用户发送的请求。

```php
　　<?php
　　　　$hash = md5($_COOKIE['cookie']);
　　?>
　　<form method=”POST” action=”transfer.php”>
　　　　<input type=”text” name=”toBankId”>
　　　　<input type=”text” name=”money”>
　　　　<input type=”hidden” name=”hash” value=”<?=$hash;?>”>
　　　　<input type=”submit” name=”submit” value=”Submit”>
　　</form>
```
然后在服务器端进行Hash值验证

```php
 <?php
　　      if(isset($_POST['check'])) {
     　　      $hash = md5($_COOKIE['cookie']);
          　　 if($_POST['check'] == $hash) {
               　　 doJob();
　　           } else {
　　　　　　　　//...
          　　 }
　　      } else {
　　　　　　//...
　　      }
      ?>
```

5. Token(主流方法)

> CSRF攻击要成功的条件在于攻击者能够预测所有的参数从而构造出合法的请求。所以根据不可预测性原则，我们可以对参数进行加密从而防止CSRF攻击。


> 另一个更通用的做法是保持原有参数不变，另外添加一个参数Token，其值是随机的。这样攻击者因为不知道Token而无法构造出合法的请求进行攻击。

### Token

#### 使用原则

> Token要足够随机————只有这样才算不可预测
> Token是一次性的，即每次请求成功后要更新Token————这样可以增加攻击难度，增加预测难度
> Token要注意保密性————敏感操作使用post，防止Token出现在URL中

##### EG：

1. 先是令牌生成函数(gen_token())：

```php
  <?php
     function gen_token() {
 　　　　//这里我是贪方便，实际上单使用Rand()得出的随机数作为令牌，也是不安全的。
　　　　//这个可以参考我写的Findbugs笔记中的《Random object created and used only once》
          $token = md5(uniqid(rand(), true));
          return $token;
     }
```

2. 然后是Session令牌生成函数(gen_stoken())：

```php
  <?php
     　　function gen_stoken() {
　　　　　　$pToken = "";
　　　　　　if($_SESSION[STOKEN_NAME]  == $pToken){
　　　　　　　　//没有值，赋新值
　　　　　　　　$_SESSION[STOKEN_NAME] = gen_token();
　　　　　　}    
　　　　　　else{
　　　　　　　　//继续使用旧的值
　　　　　　}
     　　}
     ?>
```
3.  WEB表单生成隐藏输入域的函数：　　

```php
<?php
　　     function gen_input() {
     　　     gen_stoken();
　　          echo “<input type=\”hidden\” name=\”" . FTOKEN_NAME . “\”
          　　     value=\”" . $_SESSION[STOKEN_NAME] . “\”> “;
     　　}
     ?>
```
4.  WEB表单结构：

```php
 <?php
          session_start();
          include(”functions.php”);
     ?>
     <form method=”POST” action=”transfer.php”>
          <input type=”text” name=”toBankId”>
          <input type=”text” name=”money”>
          <? gen_input(); ?>
          <input type=”submit” name=”submit” value=”Submit”>
     </FORM>
```

5. 服务端核对令牌：


参考资料：

http://www.freebuf.com/articles/web/55965.html

http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html

http://drops.wooyun.org/papers/155

https://segmentfault.com/q/1010000000377167

http://codeigniter.org.cn/forums/thread-19849-1-1.html


