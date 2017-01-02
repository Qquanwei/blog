---
layout: post
title: Javascript 数字补零
video: false
---

 有时候感觉很简单的问题都被写的很玄乎，献上我的数字补零实现
 实现数字前如果不够位数则补零,例如: 01，02

~~~
   ('0000' + yournumber).substr(-2)
~~~

