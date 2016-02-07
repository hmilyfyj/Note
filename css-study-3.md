title: CSS学习笔记
date: 2016-02-07 13:09
tags: [CSS,前端]
categories: CSS
---

假期，希望能静下心来熟悉一下前端，不奢望以此为工作 ，只是想在做网页时不会太过手足无措。

[教程地址](http://learn.shayhowe.com/html-css/getting-to-know-css/)

# Calculating Specificity(特别度)

The **type selector** has the lowest specificity weight and holds a point value of **0-0-1**. 
The **class selector** has a medium specificity weight and holds a point value of **0-1-0**. 
 The **ID selector** has a high specificity weight and holds a point value of **1-0-0**.
 As we can see, specificity points are calculated using three columns. The first column counts ID selectors, the second column counts class selectors, and the third column counts type selectors.
