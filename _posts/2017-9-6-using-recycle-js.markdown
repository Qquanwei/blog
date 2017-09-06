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

主要用于更新state, 数据源需要返回新state， 最终被setState所订阅

* dispatch(sources)

recycle 检测到使用了redux之后会去处理dispatch的数据源，
每个数据源直接返回action/function，被store.dispatch所订阅

* effects(sources)

返回由用户自定义的一个事件源， 每个事件源最终都会被订阅， 但订阅的内容将会被忽略。


这几个函数用于生成不同目的事件流， 只有effects所产生的副作用是不可预期的， 但是也是
最灵活的。

## sources

我们事件流的源头往往是基于 事件/props/timer 这些, sources就是提供了一个方便我们
订阅的对象集合。 它里面包含下面的结构

```javascript
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

其中除了`select`,`selectClass`,`selectId`之外其余的几个属性全部都是observable, 可以直接订阅。

* lifecycle

会按照实际组件生命周期产生下面三个字符串: `componentDidMount` -> `componentDidUpdate` -> `componentWillUnmount`

* state

每次render都会触发， 如有需要加上必要的distinct

* props

每次render都会触发

* store

订阅的redux store，只有store修改的时候才会触发。 该store为全局store。

## 简单介绍一下select

select用于根据元素类型进行原则，不仅支持原生的dom元素类型还支持自定义类型。 例如下面这样

```javascript
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

这里的effects中直接使用MyComponent进行选择， 然后监听其自定义事件`onSubmit`， recycle会在运行时收集所有的本view函数中渲染的组件对象， 然后找出所有select中对应的组件, 并且给其添加`onSubmit`事件。 所以虽然我们没有在view中绑定onSubmit，但是在实际渲染的时候MyComponent是已经绑定了onSubmit的。

sources提供的这几个observable基本上已经足够我们日常所需了。


## 完整的计数器例子，加减都通过服务器完成，需要前端保持事务性

由于计算是服务端完成，所以前端尽量需要保持同时只进行一次请求， 若不然会造成错乱现象。

``` javascript
import recycle from 'recycle'
import rx from 'rxjs'

// Add :: Number a => count -> a
import { Add } from './actions'

const Component = recycle({
    effects(sources) {
        const add$ = sources.selectClass('.add')
                            .addListener('onClick')
                            .mapTo(1)

        const dec$ = sources.selectClass('.dec')
                            .addListener('onClick')
                            .mapTo(-1)
        return [
            rx.race(add$, dec$)
              .withLatestFrom(sources.props)
              .exhaustMap(([count, props]) => {
                  return props.dispatch(Add(count))
              })
        ]
    },
    view (props) {
        return (
            <div>
                <h1>{ props.count }</h1>
                <div className="add">加</div>
                <div className="dec">减</div>
            </div>
        )
    }
})


function mapStateToProps (state) {
    return { count : state.count }
}
function mapDispatchToProps (dispatch) {
    return { dispatch}
}
export default connect(mapStateToProps, mapDispatchToProps)(Component)
```
