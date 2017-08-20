# Redux 入门教程（一）  ： 基本用法

一年半前， 我写了<<React 入门实例教程>> , 介绍了React的基本用法。

React 只是 DOM 的一个抽象层， 并不是Web应用的完全解决方案。 但是有两个方面，它没有涉及。

```
代码结构
组件之间的通信
```

对于大型的复杂应用来说， 这两个方面恰恰是最关键的。  因此， 只用React没法写大型应用。


为了解决这个问题， 2014年Facebook 提出了Flux架构的概念， 引发了很多的实现。 2015年，
Redux出现， 将Flux 与函数式编程结合在一起， 很短时间内， 很短时间内就成为了最热门的前端架构。

本文详细介绍Redux结构，由于内容较多，全文分成部分。

零， 你可能不需要Redux。

首先明确一点， Redux是一个有用的架构，但不是非用不可。事实上， 大多数情况，你可以不用它。只要React就够了。

曾经有人说过这样一句话。

如果你不知道是够需要Redux， 那就是不要它。

只要遇到了React实在结局不了的问题， 你才需要Redux。

简单说，如果你的UI层非常简单，没有很多互动，Redux就是不必要的，用了反而增加复杂性。

用户的使用方式非常简单。
用户之间没有协作。
不需要与服务器交互，也没有使用Websocket。
视图层只从单一来源获取数据。


上面这些情况， 都不需要使用Redux。
```

用户的使用方式复杂
不同身份的用户有不同的使用方式
多个用户之间可以协作
与服务器大量交互，或者使用websocket
view要从多个来源获取数据
```

上面这些情况才是Redux的适用场景， 多交互，多数据源。


从组件角度看，如果你的应用有以下场景， 可以考虑使用Redux。
```

某个组件的状态， 需要共享，
某个状态需要在任何地方都可以拿到
一个组件需要改变全局状态
一个组件需要改变另一个组件的状态

```

发生上面情况时，如果不使用Redux或者其他状态管理工具，不按照一定规律处理状态的读写，
代码很快就会变成一团乱麻。 你需要一种机制，可以在同一个地方查询状态，改变状态，传播状态的变化。

不要把Redux当作万灵丹， 如果你的应用还没呦那么复杂， 就没必要用它。 另一方面， Redux只是Web架构的一宗解决方案。

##  一，  预备知识

  提供一个简洁易懂的，全面的入门级参考文档
  
## 二， 设计思想很简单， 就两句话，

````
 web应用是一个状态机， 视图和状态是一一对应的。
 所有的状态， 保存在一个对象里面。
 ````

##  三，基本概念， 和API

1.  Store 

Store就是存放数据的地方，你可以把它看作一个容器， 整个应用只能有一个Store。

Redux 提供了createStore这个函数， 用来生成Store。

```
import { createStore } from 'redux';
const store = createStore(fn);
```


2.  State 

Store 对象包含所有数据， 如果想得到某个时点的数据， 就要对Store生成快照， 这种时点的数据集合，就叫做State。

当前时刻的State，可以通过store.getState()拿到。

````
import {createStore }  from 'redux';
const store = createState(fn);

const state = store.getState();

````

Redux规定，一个State对应着一个view， 只要State相同， view就相同。
你知道了State，就知道View是怎么样， 反之亦然。

3.  Action

State的变化，会导致View的变化， 但是用户接触不到State，只能接触到View。 所以State的变化必须时View导致的。
Action就是View发出的通知， 表示State应该要发生变化。

Action是一个对象。 其中的type属性必须的， 表示Action的名称。 其他属性可以自由设置，社区有一个规范可以参考。

````
const action = {
   type: 'ADD_TODO',
   payload: 'Learn Redux'
};

````
上面代码中Action的名称是ADD_TODO，  它携带的信息是字符串，Learn Redux。
可以这样理解， Action描述当前发生的事情， 改变State的唯一方法， 就是使用Action， 它会运送数据到Store。

4. Action Creator

View 要发送多少个消息， 就会有多少种Action。 如果都手写， 会很麻烦。 可以定义一个函数
来生成Action， 这个函数就叫Action Creator。



````
const Add_TODO = '添加 TODO'；

function addTOdo(text) {
    return{
       type: ADD_TODO,
       text
    }
}

````

上面代码中addTodo函数就是一个Action  Creator


5.  store.dispatch()


````

import { createStore } from 'redux';

const store = createStore(fn);

store.dispatch({
   type: 'ADD_TODO',
   payload: 'learn Redux'
});

````

上面代码中， store.dispatch 接受以一个Aciton对象作为参数，将它发送出去。


结合在Action Creator，这段代码可以改成这样。
store.dispatch(addTodo('Learn Redux'));


6.  Reducer

Store收到Action以后， 必须给出一个新的State， 这样View才会发生变化。 这种State的计算过程
叫做Reducer。

Reducer是一个函数， 它接受Action和当前State作为参数， 返回一个新的State。

const reducer = function(state, action){
    return new_state;
};

整个应用的初始状态，可以作为State的默认值， 下面是一个实际的例子。

````
const reducer = function(state, action){
  
}

````
整个应用的初始状态，， 可以作为State的默认值， 下面是一个实际的例子。

````

const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});

````

reducer函数收到名为ADD的 Action 以后，就返回一个新的 State，作为加法的计算结果。
其他运算的逻辑（比如减法），也可以根据 Action 的不同来实现




实际应用中， Reducer函数不用像上面这样手动调用， store.dispatch 方法出发Reducer的自动执行。
为此，Store需要知道Reducer函数， 做法是在生成Store的时候，将Reducer传入createStore方法。

7.   纯函数

Reducer函数最重要的特征是。 他是一个纯函数， 也就是说， 只要是同样的输入1，必定的搭配同样的输出。

纯函数是函数式编程的概念， 必须遵守以下一些约束。

````
不得改写参数，
不能调用系统的I/O 的API
不能调用Date.now 等不纯的方法，  因为每次会得到不一样的结果。
````

由于Reducer是纯函数， 就可以保证同样的State， 必定得到同样的View， 但是也正因为这一点， Reducer函数里
不能改变State，必须返回一个全新的对象， 请参考下面的写法。


````
function reducer(state, action){
return Object.assign({}, state, {thingTochange});

}
````
最好是吧state对象设置成只读， 你没法改变它， 要得到新的state， 唯一办法就是生成一个新对象。 这样的好处
是， 任何时候，与某个view对应的state总是以恶搞不变的对象。


8。  store.subscribe()

 Store 允许使用store.subscribe 方法设置监听函数， 一旦state发生变化， 就会自动执行这个函数。
 
````
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);

````

显然，只要把view的更新函数，(对于react项目来说， 就是组件的render方法， 或setState方法，)

放入listen， 就会实现view的自动渲染。


store.subscribe 方法返回一个函数， 调用这个函数就可以接触监听。
````
let unsubscribe = store.subscribe(() => 
     console.log(store.getState());
 );
 
unsubscribe();
````


##  四.  store的实现
上一节介绍了，Redux涉及的基本概念，  可以发现Store提供了三个方法，

store.getState();
store.dispatch();
store.subscribe();

createStore 方法还可以接受第二个参数， 表示State的最初状态， 这通常是服务器给出的。

Redux 提供了一个combineReducers方法， 用于Reducer的拆分， 你只要定义个各个子Reducer函数，

然后用这个方法， 将它们合成一个大的reducer。

````
import { combineReducers }  from 'redux';

const chatReducer = combineReducers({
   chatlog,
   statusMessage,
   username
});

export default todoApp;

````

这种写法有一个前提， 就是State的 属性 必须与子Reducer同名，  如果不同名，  就要采用下面的写法。

````
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})

// 等同于
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}

````

总之，看到这里，你也明白了， combineReducers 做的就是产生以一个整体的Reducer函数， 该函数根据State
key去执行相应的子Reducer，并将返回结果合并成一个大的state对象。


## 六， 工作流程

1.  用户首先发出Action。
    
     store.dispatch(action);

2.  然后Store自动调用reducer，并且传入两个参数， 当前state和收到的action， reducer会返回
    新的state。
    
    let nextState = todoApp(previousState, action);
    
    
    State 一旦发生变化， Store就会调用监听函数。
    
    store.subscribe(listener);
    
    listener可以通过store.getState()  得到当前状态， 如果使用的是React， 这时可以出发重新渲染View。
    
    ````
    
    function listener(){
       let newState = store.getState();
       component.setState(newState);
    }
    
    ````
    
## 七， 实例， 计数器，

下面我们来看一个最简单的实例。


````
const Counter = ({ value }) => (
  <h1>{value}</h1>
  <button onClick={onIncrement}>+</button>
  <button onClick={onDecrement}>-</button>
);

const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    case 'DECREMENT': return state - 1;
    default: return state;
  }
};

const store = createStore(reducer);

const render = () => {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({type: 'INCREMENT'})}
      onDecrement={() => store.dispatch({type: 'DECREMENT'})}
    />,
    document.getElementById('root')
  );
};

render();
store.subscribe(render);

````


    
    
























































































































