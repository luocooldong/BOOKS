##  
 在上一篇文章中，我介绍了Redux 的基本做法， 用户发出Action， reducer函数算出新的State，  View重新渲染界面。。

但是有一个关键问题没有解决， 异步操作怎么办？
Action发出以后， Reducer立即算出State，这叫做同步， Action发出以后， 过一段时间再执行Reducer，这叫做异步。

那么怎么才能让Reducer在异步操作结束之后自动执行呢， 这就要用到了新的工具： 中间件。middleaware

## 一， 中间件的概念，

为了理解中间件，让我们站在框架作者的角度来思考问题， 如果要添加功能，你会在哪个环节添加。

1，  Reducer： 纯函数， 只承担计算State的功能， 不合适承担起他功能， 也承担不了。 因为理论上， 纯函数不能进行读写操作。

2.  View：  与State 一一对应， 可以看作State的视觉层， 也不合适承担起他功能。

3.  Action：  存放数据的对象， 即消息的载体， 只能被别人操作， 自己也不能进行任何操作。

想来想去，  只有发送Action的这个步骤，  即store.dispatch()  方法可以添加功能，

````

let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
}

````
上面代码中， 对于store.dispatch 进行了重新定义， 在发送Action前后添加了打印功能，这就是中间件的雏形。


中间件就是一个函数， 对store.dispatch  方法进行了改造， 在发出Action和执行Reducer这两步之间，添加了其他功能。


## 二， 中间件的用法，


本章不涉及如何编写中间件，因为常用的中间件有现成的， 只要引用别人写好的模块就好，
上一节的日志中间件，就有现成的Redux-logger模块， 那么我们在这里只介绍使用中间件。

````
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const store = createStore(
  reducer,
  applyMiddleware(logger)
);

````

上面代码中， redux-logger 提供了一个生成器createLogger，可以生成日志中间件logger。
然后把它放在applyMiddleware方法之中， 传入createStore方法，这样就完成了，store.dsipatch() 的功能增强。


1.  createStore方法可以接受整个应用的初始状态作为参数，那样的话， applyMiddleware就是第三个参数了。
````
const store = createStore(
   reducer,
   initial_store,
   applyMiddleware(logger)
)
````



2. 中间件的次序有一定的讲究，。

````
const store = createStore(
    reducer,
    applyMiddleware(thunk, promise, logger)
);

````
在上面的代码中，applyMiddleware方法的三个参数， 就是三个中间件。 有的中间件有次要求， 使用前检查
一下文档。 比如logger一定要放在最后， 否则输入结果会不正确。

##  三 ，  看到这里，你可能会问， applyMiddleware 这个方法到底是在干什么的。

他是Redux的原生方法， 作用是将所有中间件促成一个数组， 依次执行，


## 。四，  异步操作的基本思路

理解了中间件之后，就可以处理异步操作了。

同步操作着急要发出一个Action即可， 异步操作的差别是它要发出三种Action。


操作发起时的Action，
操作成功时的Action，
操作失败时的Action，

以向服务器取出数据为例， 三种Action可以有两种不同的写法。

````
// 写法一：名称相同，参数不同
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }

// 写法二：名称不同
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }

````
除了Action的种类不同之外， 异步操作的State也要进行改造，反映不同的操作状态，

````

let State = {
    isFetching : true,
    didInvaildate: true,
    lastUpdated: 'XXXX'
}
````

现在整个异步操作的思路就很清楚了， 
操作开始时， 送出一个Action， 触发State更新为正在操作状态， view重新渲染。
操作结束后， 再送出一个Action， 触发State更新为操作结束状态， view再一次重新渲染。


## 。五， redux-thunk 中间件

异步操作，至少要送出两个Action， 用户出发第一个Action， 这个跟同步操作一样， 没有问题，
如何才能在操作结束之后，系统自动送出第二个Action呢。

const  fetchPosts = postTitle => (dispatch, getState) => {
    dispatch(requestPosts(postTitle));
    
    return fetch(`/some/API/${postTitle}.json`)
          .then(response => response.json())
          .then(json => dispatch(receivePosts(posTitle, json)));

};

使用方法一，
 store.dispatch(fetchPosts('reactjs));
 
 使用方法二，
  Store.dispatch(fetchPosts('reactjs)).then(() => console.log(store.getState()));
  
  ````

  1.  fetchPosts 返回了一个函数， 而普通的Action Creator 默认返回了一个对象
  2.  返回的函数的参数是dispatch 和getState 这两个Redux方法， 普通的Action Creator的参数时Action的内容
  3.  在返回的函数之中， 先发出以一个Action，表示操作开始，
  4.  异步操作结束之后， 再发出一个Action，表示操作结束
````

我们这样做虽然解决了自动发送第二个Action的问题。
但是这样又带来了另外一个问题， Action是由store.dispatch 方法发送的， 而store.dispatch 方法正常情况下，

参数只能是对象， 不能是函数。 这时，就需要使用Redux-thunk.

````
 import { createStore, applyMiddleware }  from 'redux';
 import thunk from 'redux-thunk';
 import reducer from './reducers';
 
 const store = createStore(
     reducer,
     applyMiddleware(thunk)
 );
 
````

上面使用了redux-thunk中间件， 改造了storeDispatch, 使得后者可以接受函数作为参数。

因此， 异步操作的第一个解决方案就是，写一个返回函数的Action Creator，然后使用redux-thunk, 
中间件改造store.dispatch.

##  redux-promise 中间件

既然Action Creator 可以返回函数， 当然也可以返回其他值，。
另一种异步解决方案， 就是让Action Creator返回一个Promise对象。

这就是需要使用Redux-priomise 中间件

````

import { createStore, applyMiddleware } from 'redux';
import promiseMiddleware from 'redux-promise';
import reducer from './reducers';

const store = createStore(
  reducer,
  applyMiddleware(promiseMiddleware)
); 

````

这个中间件使得store.dispatch 方法可以接受Promise 对象作为参数。
这时，Action Creator有两种写法， 返回值是一个Promise对象。
````
const fetchPosts = 

  (dispatch, postTitle) => new Promise(function (resolve, reject) {
     dispatch(requestPosts(postTitle));
     
     return fetch(`/some/API/${postTitle}.json`)
       .then(response => {
         type: 'FETCH_POSTS',
         payload: response.json()
       });
});

````

写法二， Action对象的payload 属性是一个Promise对象， 这需要从Reux-actions模块
引入createAction 方法。 

````
import { createAction } from 'redux-actions';

class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    // 发出同步 Action
    dispatch(requestPosts(selectedPost));
    // 发出异步 Action
    dispatch(createAction(
      'FETCH_POSTS', 
      fetch(`/some/API/${postTitle}.json`)
        .then(response => response.json())
    ));
  }
````


























  
  














































