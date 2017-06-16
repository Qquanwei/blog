---
layout: post
image: redux-saga.jpg
---

为了解决程序状态统一管理这个问题我们引入了redux。

我们不断 `dispatch -> reducer -> update state -> render` 这个循环，很正常不过。

但是稍微复杂一点，我们在程序中引入了异步操作，例如发起一个`http`请求，可能就会打破上面的简单流程。虽然使用`redux-thunk`可以帮我们解决这个问题，但是在真实场景下依然会面对一些不太容易解决的问题。

1. action一旦发起就无法取消。
2. 异步action的执行顺序无法确定
3. 没办法管理action本身。

针对第一个问题，举个例子，我们有一个自动补全的输入框，用户每输入一次就会发起一次请求补全的action, 那么如果我输入的很快(或者网络很慢)发起了多个请求，这时候我们显然不需要去处理老的请求数据，也就是action一旦发起了就无法取消。

第二个问题与第一个问题相似，因为网络响应无法预测，所以无法简单的统一处理多个请求。

第三个问题比较广泛，也比较通用。 例如一个特定action在处理完之前无法响应新的请求，或者一些次数限制 ...etc

为了解决日益凸显的管理action带来的冲突，将我们从大量的local state中解放出来，我选择了响应式编程(ReactiveX)。在redux中衍生出了redux-saga模式。

## 使用 redux-saga

引入middleware

~~~

import createSagaMiddleware from 'redux-saga';
import { createStore, applyMiddleware, compose } from 'redux';
import sagas from './sagas';
import reducer from 'reducers';

const sagaMiddleware = createSagaMiddleware();
const middlewares = [
  ...
  sagaMiddleware,
];

const store = createStore(reducer, applyMiddleware(...middlewares));

sagaMiddleware.run(sagas)
export default store;

~~~

saga是作为一个服务启动在程序中的，不仅仅是一个middleware, 上面的`sagaMiddleware.run(sagas)` 即启用我们的sagas的监听程序。


创建自定义saga

~~~

import { takeLatest } from 'redux-saga'
import { call, put } from 'redux-saga/effects'
import * as API from 'actions/api'
import {
  FETCH_AUTOCOMPLETE,
  FETCH_AUTOCOMPLETE_SUCCESS,
} from 'actions'
import axios from 'axios'

function fetch(params) {
  return axios.get(API.GET_AUTOCOMPLETE, {params})
}

function* fetchAutoComplete ({ payload }) {
  const data = yield call(fetch, payload)
  yield put({type: FETCH_AUTOCOMPLETE_SUCCESS, response: data})
}

export default function* () {
  yield [
    takeLatest(FETCH_AUTOCOMPLETE, fetchAutoComplete),
  ]
}

~~~

这个saga文件类似一个服务，其中的takeLatest有个内循环一直监听`FETCH_AUTOCOMPLETE`这个action, 如果监听到了就执行fetchAuthComplete。 相应的，如果在fetchAutoComplete挂起(yield处)时有一个新的action到来，takeLatest会进行自己的操作取消继续执行旧的fetchAutoComplete, 转而执行新的。

所以使用时就可以很方便地

~~~

dispatch({type: FETCH_AUTOCOMPLATE, payload: {q : 'h'}})
dispatch({type: FETCH_AUTOCOMPLATE, payload: {q : 'he'}})
dispatch({type: FETCH_AUTOCOMPLATE, payload: {q : 'hel'}})
dispatch({type: FETCH_AUTOCOMPLATE, payload: {q : 'hell'}})

~~~

真正能进行处理的只有之后一个

不过可能在前端可能会被`function*`吓到，什么鬼 ？？ 其实这就是saga能够取消action的原因，由外部执行器执行generator。

类似的redux-saga依然可以在内部很方便地进行`debounce`操作，可以减少用户网络请求。



redux-saga 把我们从大量繁琐的局部state解放出来，从而使得在使用时不需要考虑这些问题，不过saga也会有它的缺点，action处理可能没有那么清晰。

我们大量使用`pure function` & `ReactiveX` 其实都是在做一个问题，就是减少局部state, 让程序更加地函数式。
