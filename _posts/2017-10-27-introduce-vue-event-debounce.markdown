---
layout: post
title: vue-event-debounce组件介绍。
---

想到自己几个月前写的一个小插件还有点用处，准备写篇博客介绍一下。

在vue/react中一直以来有一个非常令人头疼的问题，就是事件防抖。（ok, 我了解，你会用lodash/debounce解决，但是听我说完）这个小家伙无处不在，凡是触发async action的地方几乎都要应用防抖操作。

以vue为例，你可能会这样来处理抖动

```(javascript)
<template>
  <div @click="handleTriggerAction">click me</div>
</template>
<script>
import Vue from 'vue';
import { Component } from 'vue-property-decorator';
import debounce from 'lodash/debounce';

@Component
export default class MyComponent extends Vue {
  constructor () {
    super()
    this.handleTriggerAction = debounce(this.handleTriggerAction, 100);
  }
  handleTriggerAction() {
     // send async request
  }
}
</script>
```

(vue的class component写法，anyway这不重要)。上面的写法不好，甚至根本不会达到你预想的效果，因为debounce或者throttle的限流作用非常简单，只有一个截止时间限制，我们无法预测异步action的实际时间所以这些简单的限流有很大的风险导致我们并发相同请求。

第二点是这种写法很冗余，写了一堆没用的操作, 虽然可以借助第三方decorator库来实现，但是有点不值得。


所以看下来还是[vue-event-debounce](https://github.com/Qquanwei/vue-event-debounce)比较好，最终的size才1.3k。 vue-event-debounce 的原理也很简单，就是增加一个指令v-click来实现。

```(javacsript)
<template>
  <div v-click="handleTriggerAction">click me</div>
</template>
<script>
import Vue from 'vue';
import { Component } from 'vue-property-decorator';
import debounce from 'lodash/debounce';

@Component
export default class MyComponent extends Vue {
  handleTriggerAction() {
    return someAsyncActiono()
  }
}
</script>
```

使用之后v-click会判断事件的返回值是否是promise类型，如果不是则同步执行，和普通事件一样。
如果是promise则会等待这个promise执行结束才会允许接受下一次的事件。

非常简单清晰地安全执行异步action, 感兴趣的可以看下源码: https://github.com/Qquanwei/vue-event-debounce/blob/master/src/index.js
