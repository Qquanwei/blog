---
layout: post
title: vue-apollo 使用
video: false
---

最近有段时间没有更新博客了，这段时间由于要做一个html5个项目使用到了vue-apollo感觉习惯了，使用其他觉得还不错。

`vue-apollo`是给vue封装的一个apollo插件，不过没有`react-apollo`那么出名, 但使用原理很相似，我觉得要想使用好apolloclient就要首先理解它的工作原理，以及与传统RESTful有什么区别。 ApolloClient是一个高度封装的graphql框架, 将网络+存储都封装到了一起,也正是这种高度集中带给我们的好处是可以将网络与state去耦合到前端框架中，我们只需在组件中建立一种数据依赖就可以让apollo自动工作起来。

我们组件的更新是依赖状态的更新，我们的query也是依赖variables的更新, 我们不需要手动去执行何时去query数据从服务器上，而是预先设定好依赖，当component中状态更新时apolloclient会自动检查依赖是否发生变更，如果变更了则去重新请求数据源。看一下传统的写法

~~~
<template>
  <div>
    <h3> `{{ data }}`</h3>
    <input v-model="myInput" type="number" placeholder="input some number"/>
  </div>
</template>

<script>
  improt myAction from './action.js'
  export default {
    data () {
      return {
        myInput: ''
      }
    },
    watch: {
      myInput (input) {
        myAction(input).then((result) => {
          this.data = result
        })
      }
    }
  }
</script>
~~~

当用户输入的数字后去请求数字对应的结果，不要追究为什么这样做，假设我们有这个需求就好。这时候就是传统的写法，后端提供一个RESTFul的API，前端去手动根据条件发起请求。一切都是那么的合乎常理。 不过下面看一下使用vue-apollo的写法

~~~
<template>
  <div>
    <h3> `{{ myQuery.data }} `</h3>
    <input v-model="myInput" type="number" placeholder="input some number"/>
  </div>
</template>

<script>
  import myQuery from './querys.js'
  export default {
    data () {
      return {
        myInput: '',
        myQyert: {
          data: ''
        }
      }
    },
    apollo: {
      myQyery: {
        query: myQuery,
        variables () {
          return {
            number: this.myInput
          }
        },
        skip () {
          return !this.myInput
        }
      }
    }
  }
</script>
~~~

ok ， 完事。 看上去似乎我们的代码变得更长了，更复杂了，实际上确实，因为要建立一个依赖模型需要写一些额外的代码,但是好处就是我们的组件是自动工作的！这里第一个demo没有加上缓存机制，实际上如果加上缓存第一个代码也会变得更加复杂，但是在apolloclient里缓存也是自动完成的。例如如果我们输入了相同的数字，那么apolloclient将直接从state里去拿数据，而不会发起请求,因为依赖没有发生变化! 而且我们甚至不用去使用第三方的ajax库，vue-apollo里自带了一个fetch的polyfill来享受新版本浏览器带来的便利。当然vue-apollo还有其他的一些选项来完成更加复杂的场景，不过上面的代码让我们看到了一丝希望，就是网络与状态全部都是自动化完成！ great

实际上vue-apollo/react-apollo 都给我们提供了相同的一套机制，(但是vue-apollo起步较晚，直到前几天的几次更新才让它相对更加完善一些), 让我们摆脱了RESTFul给我们的schema限制。还有一个额外的好处是由于graphql是自文档的，所以也减少了人工沟通的成本，人工沟通的成本又往往是巨大的~
