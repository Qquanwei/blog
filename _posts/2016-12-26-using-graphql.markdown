---
layout: post
title: Apollo-Client 使用
---

 这几天尝试使用graphql重构一下代码,graphql的优势是可以前后端都有一套schema让前端不再受限于后端的数据结构变动,了解了一下`apollo-client` , apollo-client 工作流程如下

![Apollo work flow]({{site.baseurl}}/content/images/apolloclient.png)

 `apollo-client`的内部状态也是由`redux`进行管理，并且也加了一层normalize处理，可以简化数据更新修改操作 .

 开始在react中使用apollo-client

首先是安装

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


graphql还有一个很不错的地方就是多个query可以一起查询，通过一个请求获取到所需要的所有数据.