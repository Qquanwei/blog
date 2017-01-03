---
layout: post
title: Apollo-Client 使用
---

 这几天尝试使用graphql重构一下代码,graphql的优势是可以前后端都有一套schema让前端不再受限于后端的数据结构变动,了解了一下`apollo-client` , apollo-client 工作流程如下

![Apollo work flow]({{site.baseurl}}/content/images/apolloclient.png)

 `apollo-client`的内部状态也是由`redux`进行管理，并且也加了一层normalize处理，可以简化数据更新修改操作 .

 开始在react中使用apollo-client

## 安装

~~~javascript
 npm install apollo-client graphql-tag react-apollo --save
~~~

main.jsx

~~~javascript
import React from 'react';
import { render } from 'react-dom';
import ApolloClient ,{ createNetworkInterface } from 'apollo-client';
import {ApolloProvider} from 'react-apollo';
import {Router,hashHistory} from 'react-router';
import Page from './page.jsx;
import WelCome from './welcome.jsx;

const client = new ApolloClient({
    networkInterface: createNetworkInterface({uri : 'http://localhost:3000/graphql'})
});

const routes={
  component: Page,
  path: '/',
  indexRoute:{
    component: WelCome
  },
  childRoutes:[{
    component: Me,
    path: '/me'
  }]
}
render(<ApolloProvider client={client}>
         <Router routes={routes} history={hashHistory}/>
        </ApolloProvider> ,document.getElementById('app'))

~~~


一个很简单的React程序结构, 首先因为我们需要让Apollo来管理数据状态，所以使用自带的`ApolloProvider`来作为顶层组件. `createNetworkInterface`还有一个有用option是 `dataIdFromObject` , 为一个函数类型，用来normalize我们的数据用,如果不提供的话那么默认的normalize的特性就不能用，所以如果后端数据库是mongo的话，可以简单地设置`dataIdFromObject: (j)=>j._id` 来使用normalize特性.

## Query

page.jsx

~~~javascript
import React from 'react';

class Page extends React.Component{
  render(){
    return (
      <div>
      	{ this.props.children } 
      </div>
    )
  }
}

export default Page;
~~~

welcome.jsx

~~~javascript
import React from 'react';
import gql from 'graphql-tag';
import { graphql } from 'react-apollo';

@graphql(gql`query query{ welcome { text} }`)
class WelCome extends React.Component{
  render(){
    if(this.props.data.loading)
      return <div>loading</div>;
    return <div>{ this.props.data.welcome.text }</div>
  }
}
export default WelCome;
~~~

welcome组建的作用是从后端获取欢迎文字然后显示,这里使用了decorator特性,可以安装`babel-plugin-transform-decorators-legacy`来获取该特性(别忘了配置一下babel)或者也可以下面这种形式

~~~javascript

  export default graphql(gql`query query{ welcome { text }}`)
                    (WelCome);
  
~~~

如果我们想构建一个带有参数的查询也可以像下面这样

~~~
@graphql(gql`query query($name:String){ welcome(name:$name) { text} }`,{
  options: ownProp=> ({variables: {name: ownProp.name || 'guest'}})
})
class WelCome extends React.Component{
 ...
}
~~~


请求操作将会随着组建的挂载而自动执行, 默认的数据将会在组件的props中data下,data字段如下

~~~
 {
   ...fields //query的root字段
   loading: bool
   error: array 错误信息
   ... //其他字段
 }
~~~



## mapStateToProps

我们请求到的数据最终会放在redux进行存储，可是如果我们打印apollo的state看到的确是一大堆normalized处理过的数据，我们想在其他组件`mapStateToProps`的时候想将一些数据传入到组件中的时候该怎么做？ 传统的方法是先denormalized这些数据然后挂载到组件上, 但是性能是个硬伤。Apollo给我们提供的是另外一个思想，就是数据一旦通过query获取到本地就认为这个数据是不变的,除非你自己调用了某些motation方法然后自己手动`updateQuery`修改本地的state，否则这个state就认为是一直不变的。 所以，当我们在一个组件中query到数据之后，其他组件再次发起相同的query的时候是直接从本地拿的数据! 这也就是说我们如果想要在组件间共享我们的state，只需要在每个组件写相同的query即可,虽然对于含有参数的query还是有点不方便。如果想强制更新本地的state,query的时候带上`forceUpdate:true` , 或者使用`pollInternal`定时刷新本地数据。


## 总结
目前`apollo-client`给我的感觉是做了很多工作，网络层和数据层都是它来完成，我们只需要处理好逻辑层和我们的view, 思维需要从原来的全部都自己做转换过来，所以会有点不适应。 但是一旦适应了这一套方案之后在新需求到来的时候将会比原来的方案要好很多。