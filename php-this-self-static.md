title: PHP:this、self、static的区别
date: 2016-01-16 11:07
tags: [PHP]
categories: PHP 
---

### this、self、static

    <?php
    header("content-type:text/html;charset=utf-8");
    class Human{
     static public $name = "小妹";
     public $height = 180;
     static public function tell(){
     echo self::$name;//静态方法调用静态属性，使用self关键词
     //echo $this->height;//错。静态方法不能调用非静态属性
    //因为 $this代表实例化对象，而这里是类，不知道 $this 代表哪个对象
     }
     public function say(){
     echo self::$name . "我说话了";
     //普通方法调用静态属性，同样使用self关键词
     echo $this->height;
     }
    }
    $p1 = new Human();
    $p1->say(); 
    $p1->tell();//对象可以访问静态方法
    echo $p1::$name;//对象访问静态属性。不能这么访问$p1->name
    //因为静态属性的内存位置不在对象里
    Human::say();//错。say()方法有$this时出错；没有$this时能出结果
    //但php5.4以上会提示
    ?>
#### 结论：

> （1）、静态属性不需要实例化即可调用。因为静态属性存放的位置是在类里，调用方法为"类名::属性名"；
> （2）、静态方法不需要实例化即可调用。同上 （3）、静态方法不能调用非静态属性。因为非静态属性需要实例化后，存放在对象里；
> （4）、静态方法可以调用非静态方法，使用 self 关键词。php里，一个方法被self:: 后，它就自动转变为静态方法；

[原文链接](http://www.jb51.net/article/60871.htm)
