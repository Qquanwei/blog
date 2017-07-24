---
layout: post
title: why xstream
image: xstream.jpeg
---

当我将RxJS整体打包到bundle中的时候发现体积多了200+Kb, 对于不是很大的应用来说负担确实已经很重了。然后我就把目光转移到[xstream](https://github.com/staltz/xstream)上了。

xstream好处:

1. 精简的operator:

rxjs5发布的时候已经有了175+个operator, 根据二八原则有大部分是我们用不到的，所以`André Staltz`(xstream作者) 做了一项问卷调查发现大约有26个poerator是常用的, 例如`map`, `filter`,`combine`,`take`等, 这些常用的operator被集成到了xstream默认operator中, 从而精简了大量operator带来的噪音。 如果需要更多的operator可以在`xstream/extra`中找到.

2. 更加符合直觉的命名

RxJS 与Rx.NET是一脉相承的，所以命名上是相统一的，但是这些命名对于之前没有接触过Rx的初学者并不是特别友好，所以作者丢掉了历史包袱直接给xstream起了一套新的名字，但实际使用中并没有造成太大的麻烦，如果你已经了解了rxjs 可以读一读这两者的差异部分: https://github.com/staltz/xstream/wiki/Migrating-from-RxJS

3. 体积小

这点不用过多介绍了，(rxjs)200+kb -> (xstream)30+kb, 如果项目对于体积比较敏感这将是一个很大的优势。


TL;DR:

xstream诞生是因为在cycle.js中需要一种更加轻量的响应式框架, 以及更加容易使用和对新手更友好, 相对而言xstream上手比较容易(例如rxjs中combineLatest被rename成combine), 但是矛盾的是xstream的文档部分略有不足。
