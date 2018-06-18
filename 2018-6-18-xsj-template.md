---
title: 小书匠模板兼容  Hexo
date: 2018-06-18 10:48:51
tags: 笔记,小书匠
categories: 小书匠
---

小书匠是我用来写笔记的工具，它可以直接同步笔记到 Github，进而触发 `travis ci` 构建渲染进程。

<!-- more -->

---

```
---
<%
// 对Date的扩展，将 Date 转化为指定格式的String
    // 月(M)、日(d)、小时(h)、分(m)、秒(s)、季度(q) 可以用 1-2 个占位符， 
    // 年(y)可以用 1-4 个占位符，毫秒(S)只能用 1 个占位符(是 1-3 位的数字) 
    Date.prototype.Format = function (fmt) { //author: meizz 
        var o = {
            "M+": this.getMonth() + 1, //月份 
            "d+": this.getDate(), //日 
            "h+": this.getHours(), //小时 
            "m+": this.getMinutes(), //分 
            "s+": this.getSeconds(), //秒 
            "q+": Math.floor((this.getMonth() + 3) / 3), //季度 
            "S": this.getMilliseconds() //毫秒 
        };
        if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
        for (var k in o)
            if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
        return fmt;
    }
%>title: <% print((new Date()).getFullYear().toString()+ '-'+ ((new Date()).getMonth() + 1).toString() + '-'+ (new Date()).getDate().toString()); %>未命名文件 
date: <% print((new Date()).Format("yyyy-MM-dd hh:mm:ss")); %>
tags: Nginx
categories: Nginx
---


<!-- more -->

---
```