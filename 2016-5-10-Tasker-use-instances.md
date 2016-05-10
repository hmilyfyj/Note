title:  巧用 Tasker 自动复制验证码
date: 2016-05-10 19:03
tags: [折腾,Tasker,Android]
categories: Android
---

[Tasker:Android系统增强神器](http://coolapk.com/apk/net.dinglisch.android.taskerm)

>Tasker绝对称得上是Android系统的神器之一，与Auto Memory Manager不同，Tasker不是加速型的软件，而是系统增强型的软件，由于有众多系统状态可控制，故使得Tasker一跃成为Android系统中最闪亮的明星。但Tasker也无疑是最难使用的软件，由于可以控制的地方太多，反而让人觉得有些无所适从，不知道要从哪开始下手

[中文文档](http://tasker.dinglisch.net/userguide/zh/index.html)


<!-- more -->

---

# 知识点

# 实例

## 手机自动复制验证码

以下内容保存为 `自动复制验证码.prf.xml ` 并放入手机的 `Takser/profile`  目录，打开 App 导入即可。
```xml
<TaskerData sr="" dvi="1" tv="4.8m">
    <Profile sr="prof37" ve="2">
        <cdate>1411872904749</cdate>
        <edate>1460293109606</edate>
        <id>37</id>
        <mid0>38</mid0>
        <nme>自动复制验证码</nme>
        <Event sr="con0" ve="2">
            <code>7</code>
            <pri>0</pri>
            <Int sr="arg0" val="2" />
            <Str sr="arg1" ve="3" />
            <Str sr="arg2" ve="3" />
        </Event>
    </Profile>
    <Task sr="task38">
        <cdate>1411872909693</cdate>
        <edate>1460292990573</edate>
        <id>38</id>
        <pri>100</pri>
        <Action sr="act0" ve="7">
            <code>37</code>
            <on>false</on>
            <ConditionList sr="if">
                <Condition sr="c0" ve="3">
                    <lhs>%SMSRB</lhs>
                    <op>2</op>
                    <rhs>*码*</rhs>
                </Condition>
            </ConditionList>
        </Action>
        <Action sr="act1" ve="7">
            <code>129</code>
            <Str sr="arg0" ve="3">var text =global('SMSRB'); var m=text.match(/(验证码|校验码|动态码|确认码|随机码|验证|校验|验证密码|动态密码|校验密码|随机密码|确认密码|激活码|兑换码|认证码|认证号码|认证密码|交易码|交易密码|授权码|操作码|密码|提取码|安全代码).*?(\d{4,6})/); //flash(text) if(m){ setClip(m[2]) flash(m[0]) }
            </Str>
            <Str sr="arg1" ve="3" />
            <Int sr="arg2" val="1" />
            <Int sr="arg3" val="45" />
        </Action>
    </Task>
</TaskerData>

```