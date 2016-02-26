
title: JavaScript笔记：原型和闭包
date: 2016-02-26 19:39
tags: [JavaScript,在成为最厉害最厉害最厉害的路上]
categories: JavaScript

---
 
每天进步一点点。

---
<!-- more -->

# 对象

```javascript
function show(x) {

    console.log(typeof(x));    // undefined
    console.log(typeof(10));   // number
    console.log(typeof('abc')); // string
    console.log(typeof(true));  // boolean

    console.log(typeof(function () { }));  //function

    console.log(typeof([1, 'a', true]));  //object
    console.log(typeof ({ a: 10, b: 20 }));  //object
    console.log(typeof (null));  //object
    console.log(typeof (new Number(10)));  //object
}
show();
```

以上代码列出了`typeof`输出的集中类型标识，其中上面的四种（`undefined`, `number`, `string`, `boolean`）属于简单的**值类型**，不是对象。剩下的几种情况——**函数**、**数组**、**对象**、**null**、**new Number(10)**都是**对象**。他们都是**引用类型**。


判断一个变量是不是对象非常简单。值类型的类型判断用typeof，引用类型的类型判断用instanceof。

```javascript
var fn = function () { };
console.log(fn instanceof Object);  // true
```

对象，即若干属性的集合。

javascript的对象相对C、JAVA比较随意——数组是对象，函数是对象，对象还是对象。对象里面的一切都是属性，只有属性，没有方法。那么这样方法如何表示呢？——方法也是一种属性。因为它的属性表示为键值对的形式。

![enter image description here](http://box.kancloud.cn/2015-09-21_55ff97ea87dbc.png)

这个可能比较好理解，那么函数和数组也可以这样定义属性吗？——当然不行，但是它可以用另一种形式，总之函数/数组之流，只要是对象，它就是属性的集合。

以函数为例子：

```javascript
var fn = function () {
    alert(100);
};
fn.a = 10;
fn.b = function () {
    alert(123);
};
fn.c = {
    name: "王福朋",
    year: 1988
};
```

实例：

```javascript
console.log(typeof ($));  // function
console.log($.trim(" ABC "));
```
在jQuery源码中，`jQuery`或者$，这个变量其实是一个函数。`$.trim()` 也是个函数，它就是在$或者jQuery函数上加了一个trim属性，属性值是函数，作用是截取前后空格。


# 函数

apply

>指定函数的this指向哪个对象，可以用函数本身的apply方法，它接收两个参数，第一个参数就是需要绑定的this变量，第二个参数是Array，表示函数本身的参数。

example：

```javascript
function getAge() {
    var y = new Date().getFullYear();
    return y - this.birth;
}

var xiaoming = {
    name: '小明',
    birth: 1990,
    age: getAge
};

xiaoming.age(); // 25
getAge.apply(xiaoming, []); // 25, this指向xiaoming, 参数为空
```


