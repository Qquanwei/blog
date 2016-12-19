---
layout: post
title: Ramda,从开始到重构
video: false
image: ramda.png
---
函数式编程是最近人们开始尝试使用的一种编程方式，在[awesome-javascript](https://github.com/sorrycc/awesome-javascript#functional-programming)上列出了常见的Javascript函数式编程库,主要有： `underscore` , `lodash` , `Sugar` , `lazy.js`,`ramda`,`mount`,`mesh`,`preludejs`

之前因为没接触其他的，使用的第一个就是ramda，感受到的就是ramda的工具方法设计的特别棒，例如：所有函数都是无副作用的,所有方法都是自动柯里化的。导致写出来的代码具有非常高的组合度。

举个栗子看下(`R = require('ramda')`)
-----------------
从一个array中取出`name:'jak'`的对象

~~~~~~
  R.find(R.propEq('name','jak'),myarray)
~~~~~~

传统的写法为

~~~~~~
  myarray.find((item)=> item.name === 'jak')
~~~~~~

可以看到传统的写出的代码有点冗余，写了一些没用的辅助代码，而使用ramda的看起来可读性就很好。

再看一个复杂点的例子

取出`name.lastname`属性为`'jak','Jonh','M'`的对象。原生写法如下

~~~~~~
  myarray.filter( (item)=>{
     const lastname = item.name.lastname;
     return lastname === 'jak' ||
            lastname === 'jak' ||
            lastname === 'M';
  })
~~~~~~
{: .language-javascript}

看下ramda的写法

~~~~~~
   R.filter(
      R.compose(
         R.contains(R.__, ['jak','Jonh','M']),
         R.path(['name','lastname'])),
      myarray)
~~~~~~
{: .language-javascript}

如果不是很了解ramda看起来就比较恐怖，下面来拆分一下ramda的求值过程。

`R.compose`意思是组合,相当于管道的意思,前一个操作的结果是后一个操作的数据,但是与`R.pipe`不同的是这个管道是从右向左进行数据流动的

`R.contains`是一个判读元素是否在array中,返回一个bool值

`R.__` 是一个占位符，可以让一个函数只传入传入部分参数,由于ramda是自动柯里化的，所以会由被占位的函数返回一个新的函数来接收被占位的数据。

`R.path` 是从一个属性路径中取出一个值

根据上面的语义我们的代码就更像是[声明式代码](https://zh.wikipedia.org/wiki/%E5%AE%A3%E5%91%8A%E5%BC%8F%E7%B7%A8%E7%A8%8B)，整个表达式的意思就是，过滤myarray,条件是满足 name.lastname 的值在['jak','Jonh','M'] 中。

ramda提供了丰富的逻辑操作非常适用于描述一段逻辑，而且ramda的函数无副作用所以不用担心源数据被污染。在写ramda的过程中，脑子里想的不是【这段代码如何实现?】而是 【我要实现什么功能】,而一旦适应后者就再也回不到正常的编码过程中了，恨不得所有逻辑都用ramda重写一遍!



不过其实ramda带给我的更大的是~~~ 写出的一段很长的ramda代码睡一觉后没人能看得懂...














