---
layout: post
title: recycle 初步使用介绍。
---

recycle是react+rxjs的一个组件类,灵感来源于cycle, 目的是提供了一个基于事件流的组件。 在cycle中一切的数据源都是observable, recycle由于是兼容原始react组件所以没那么严格， 只是将数据源封装的更像是一个observable。


```javascript
import recycle from 'recycle'

export default recycle({
   initialState: {
     count: 0
   },
   update (sources) {
      return [
        sources.select('button')
               .addListener('onClick')
               .reduce((state, event) => ({...state, count: state.count + 1}))
      ]
   },
   view (props, state) {
      return (
        <div>
           <button> click me { state.count }</button>
        </div>
      )
   }
})
```

上面是个简单的计数器例子。 `initialState` 是初始state值,`update` 用于生成事件流,
`view`类似我们的render函数。

除此之外recycle 提供了下面几个用于不同目的函数

* update(sources)

主要用于更新state, 数据源最终返回的是新state， 最终被setState所订阅

* dispatch(sources)

recycle 检测到使用了redux之后会去处理dispatch的数据源，
每个数据源直接返回action，被store.dispatch所订阅

* effects(sources)

返回由用户自定义的一个事件源， 每个事件源最终都会被订阅， 但订阅的内容将会被忽略。


这几个函数用于生成不同目的事件流， 只有effects所产生的副作用是不可预期的， 但是也是
最强大的， 例如在处理一个thunk时简单使用dispatch往往达不到我们的效果。

## sources

我们事件流的源头往往是基于 事件/props/timer 这些, sources就是提供了一个方便我们
订阅的对象集合。 它里面包含下面的结构

```
{
  select,
  selectClass,
  selectId,
  lifecycle,
  state,
  props,
  store // 如果使用了redux的话
}
```

其中除了`select`,`selectClass`,`selectId`之外其余的几个属性全部都是observable, 可以
直接订阅。

简单介绍一下select:

select用于根据元素类型进行原则，不仅支持原声的dom元素类型还支持自定义类型。 例如下面这样

```
import MyComponent from 'mycomponent'

export default recycle({
  effects(sources) {
    return [
      sources.select(MyComponent)
             .addListener('onSubmit')
             .mapTo('submit')
    ]
  },
  view (props, state) {
     return (
        <div>
          <MyComponent/>
        </div>
     )
  }
})

```

在effects中我们直接使用MyComponent进行选择， 然后监听其自定义事件`onSubmit`， recycle会在运行时收集所有的本view函数中渲染的组件对象， 然后找出所有select中对应的组件, 并且给其添加`onSubmit`事件。 所以虽然我们没有在view中绑定onSubmit，但是在实际渲染的时候MyComponent是已经绑定了onSubmit的。

由sources提供的这几个observable已经足够我们日常所需了。


## 一些实际例子
