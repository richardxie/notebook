## React
### 概念
- 虚拟DOM
   每次事件循环完成后， 通过diff算法比较出虚拟DOM与真实DOM的差异，更新Dirty的DOM

- 组件（ Component ）
    - 组件属性
        属性不变
    - 组件状态
        将组件看成是一个状态机，一开始有一个初始状态，然后用户互动，导致状态变化，从而触发重新渲染 UI 
    - Render
        render方法防治渲染组件
- 受控组件
   组件中有输入组件，比如Input， Select等等，由于其会改变state的值破坏了React的数据流，因此引入受控组件。
   受控组件不显示直接输入信息，而是通过监控state，通过改变state的值最终显示输入的信息
- 非受控组件
    通过Ref直接获取原生的控件
    
### JSX

JSX是快速生成react元素的一种语法，实际是React.createElement(component, props, ...children)的语法糖，同时JSX也是Js的语法扩展，包含所有Js功能。

> const element1 = (  
>
> ​	<h1 className="greeting">    Hello, world!  {this.props.user.name}  </h1> ); 
>
> // 等价 
>
> const element2 = React.createElement(  'h1',  {className: 'greeting'},  'Hello, world!'  ...);

### 容器组件与展示组件

- 容器组件
    容器组件需要知道如何获取子组件所需的数据以及这些数据的处理逻辑，并把数据和逻辑沟通过props提供给子组件使用。容器组件一般是有状态组件。
- 展示组件
    展示UI，其不关心数据如何获取及数据如何修改，只需要这些数据如何展示，UI是什么样子。父级组件通过Props传递给展示组件所需的数据及修改数据的回调函数。展示组件一般是无状态组件。如果有受控组件，是可以使用state，这个state属于UI state。负责UI 状态的改变时的展示。
    
```TypeScript
class UserPage extends React.Component {
    constructor(props){
        super(props);
        this.state = {
            users:[]
        }
    }
    componentDidMount() {
        var that = this;   
        fetch('/path/to/user-appi').then(function(response) {               response.json().then(function(data) {       
                that.setState({users: data})    
            });  
        });
    }
    render() {
        return (<Users users = {this.stata.users} />)
    }
}

function Users(props) {
    return (<div>
            <ul className="user-list">       
            {
                    props.users.map(function(user) {         
                        return (           
                            <li key={user.id}>             
                                <span>{user.name}</span>           
                            </li> );       
                    })
            }     
            </ul>
            </div>)
}

```

### 属性与状态
- 属性（Props）组件对外的接口，属性是不变的。
- 状态 （State）组件对内的接口，状态是可变的。
    - 从父组件通过Props传过来的变量，不是状态
    - 组件生命周期类不变的变量，不是状态
    - 通过其他属性或状态计算得到的变量，不是状态
    - render方法中不使用的变量，不是状态，可以定义为组件属性

    - 改变状态
        - 不能通过直接修改state的值改变状态，必须通过setState改变并且setState是异步操作
        - 并发情况下，增量修改状态
        
           ```javascript
           this.setState((perState, props)=> {
                counter: preState.counter + 1;
           })
           ```
           
        - 浅拷贝
        


### 事件
- 使用匿名函数
    - 箭头函数
    <button onClick= {() => {console.log("clicked:" + this)}}>
    箭头函数解决了this绑定的问题，this指向函数的this
    - 普通函数
    <button onClick= {function(){console.log("clicked:" + this)}}>
    this指向运行时调用函数的对象
- 使用组件
    ```typescript
    class Component1 extends React.Componet {
        constructor(props) {
            super(props)
            this.handleClick = this.handleClick.bind(this)
        }
        
        handleClick() {
            this.setState({number: ++this.state.number});
        }
        
        render() {
            return (<button onClick={this.handleClick} />)
        }
    }
    ```
    
    - 使用属性
    ```javascript
     class Component1 extends React.Componet {
        constructor(props) {
            super(props)
            this.handleClick = this.handleClick.bind(this)
        }
        
        handleClick= () => {
            this.setState({number: ++this.state.number});
        }
        
        render() {
            return (<button onClick={this.handleClick} />)
        }
    }
    
    ```
    
- 其他

### 安装调试

- 使用淘宝镜像
  
    - npm install -g cnpm --registry=https://registry.npm.taobao.org
    
        cnpm -v; 
    
        cnpm config set registry https://registry.npm.taobao.org
    
- create-react-app安装

    - cnpm install -g create-react-app
    - create-react-app my-app
        - npm start //Starts the development server
        - npm run build //Bundles the app into static files for production
        - npm test //Starts the test server
        - npm run reject //Remove this tool and copies build dependencie

- react/redux devtools 
  
    - chrome插件安装 将下载的zip文件拖到chrome://extensions/
    
    - firefox安装 addons
    
    - yarn add redux-devtools-extension --dev
    
        ```javascript
        import { composeWithDevTools } from 'redux-devtools-extension';
        const composeEnhancers = composeWithDevTools({
         // Specify name here, actionsBlacklist, actionsCreators and other options if needed
        });
        const store = createStore(
         tasks,
         composeEnhancers(applyMiddleware(thunk))
        );
        ```
    

## Redux

redux 是 js 应用的可预测状态的容器。 可以理解为全局数据状态管理工具（状态管理机），用来做组件通信等。

### 概念
- state
    - 状态数据，普通的JS对象，只能通过action来修改。
    ```javascript
    interface ReduxState {num: number}
    ```

- action
    -  动作数据，是一个简单对象用于描述state发生了什么。
    -  同步action
    ```javascript
    const INCREMENT = 'INCREMENT'
    const incrementAction = {"type": INCREMENT, "count": 2}
    
    interface Action {
        type: string,
        count:  number
    }
    
    ```
```
    
    - 异步action（redux-thunk）
    
        用于支持dispatch函数
    
        ```js
        const asyncAction = dispatch => {
            api.createTask({title, description, status})
            	.then(resp => {dispatch(createTaskSucceeded(resp.data))})
        	};
        
        
        }
```


​    
-  reducer
   
    -  动作， 根据action对state进行操作
    
    ``` javascript
    
    const initData = { num: 0 }
    const calculate = (state: ReduxState = initData, action: Action ) => {
    switch (action.type) {
        case INCREMENT:
            return {num: state.num + action.count}
        case REDUCE:
            return {num: state.num - action.count}
        default:
            return state
}}
    
    export {calculate}
    ```
    
- store
    store就是整个项目保存数据的地方，并且只能有一个。创建store就是把所有reducer给它.
    ```javascript
    
    import { createStore, combineReducers } from "redux";
    import { calculate } from "./calculate";

    const rootReducers = combineReducers({calculate})
    export const store = createStore(rootReducers)
    
      
   store.getState(); //读取store的当前状态
   store.dispatch({type: 'FETCH_TASK', payload:{}}) //向Reducer发送Action来更新Store
   store.,subscribe(() => { console.log('current state:', store.getState())}) //store监听器，当store改变时动作
   ```
   
- dispatch
store.dispatch()是组件发出action的唯一方法。
  
    ```javascript
    store.dispatch(incrementAction);
    ```
  
- middleware

  位于action（正在派发）和store（用于将action传递到reducer并广播更新状态）之间的代码。
  
    

![](img\redux.jpg)

### 原则

- 唯一数据源
    Redux 只用唯一一个 store 储存应用所有的 state，被称为 single source of truth（唯一数据源）。store 中的数据结构往往是个给应用使用的深度嵌套的对象
- State 是只读的
    惟一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。
- 使用纯函数来执行修改
    Reducer 只是一些纯函数，它接收先前的 state 和 action，并返回新的 state

### 副作用

指与Redux应用之外的世界进行的所有交互，包括服务器或本地缓存交互。

- action的创建器可以处理副作用

  ```javascript
  export function createTask({title, description, status = 'Unstarted'}) {
  	return dispatch => {
  		api.createTask({title, description, status}) //服务器交互
  			.then(resp => {dispatch(createTaskSucceeded(resp.data))}) //异步action更新store
  	};	
  }
  ```

  

- middleware可以处理副作用

  ```javascript
  export function fetchTasks() {
      return {
          [CALL_API]: { //API中间件处理API调用的流程与副作用
              types: [FETCH_TASKS_STARTED, FETCH_TASKS_SUCCEEDED, FETCH_TASKS_FAILED],
              endpoint: '/tasks'
          }
   	}
  }
  ```

  - redux-saga

    > `redux-saga` 是一个用于管理应用程序 Side Effect（副作用，例如异步获取数据，访问浏览器缓存等）的 library，它的目标是让副作用管理更容易，执行更高效，测试更简单，在处理故障时更容易。
    >
    > 可以想像为，一个 saga 就像是应用程序中一个单独的线程，它独自负责处理副作用。 `redux-saga` 是一个 redux 中间件，意味着这个线程可以通过正常的 redux action 从主应用程序启动，暂停和取消，它能访问完整的 redux state，也可以 dispatch redux action。

    ```javascript
    import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
    import Api from '...'
    
    // worker Saga : 将在 USER_FETCH_REQUESTED action 被 dispatch 时调用
    function* fetchUser(action) {
       try {
          const user = yield call(Api.fetchUser, action.payload.userId);
          yield put({type: "USER_FETCH_SUCCEEDED", user: user});
       } catch (e) {
          yield put({type: "USER_FETCH_FAILED", message: e.message});
       }
    }
    
    /*
      在每个 `USER_FETCH_REQUESTED` action 被 dispatch 时调用 fetchUser
      允许并发（译注：即同时处理多个相同的 action）
    */
    function* mySaga() {
      yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
    }
    
    /*
      也可以使用 takeLatest
    
      不允许并发，dispatch 一个 `USER_FETCH_REQUESTED` action 时，
      如果在这之前已经有一个 `USER_FETCH_REQUESTED` action 在处理中，
      那么处理中的 action 会被取消，只会执行当前的
    */
    function* mySaga() {
      yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
    }
    
    export default mySaga;
    ```

### 样例

- 复合Reducer
    ```javascript

    import { createStore, combineReducers } from 'redux';

    // The User Reducer
    const userReducer = function(state = {}, action) {
      return state;
    }

    // The Widget Reducer
    const widgetReducer = function(state = {}, action) {
      return state;
    }

    // 合并 Reducers
    const reducers = combineReducers({
      userState: userReducer,
      widgetState: widgetReducer
    });

    const store = createStore(reducers);
    ```
    
    

## React-Redux

![react-redux](img\react-redux.jpg)

连接React Component 与 Redux State

```
import React from 'react';
import { connect } from 'react-redux';
import store from '../path/to/store';
import axios from 'axios';
import UserList from '../views/list-user';

const UserListContainer = React.createClass({
  componentDidMount: function() {
    axios.get('/path/to/user-api').then(response => {
      store.dispatch({
    type: 'USER_LIST_SUCCESS',
        users: response.data
      });
    });
  },

  render: function() {
    return <UserList users={this.props.users} />;
  }
});

const mapStateToProps = function(store) {
  return {
    users: store.userState.users
  };
}

export default connect(mapStateToProps)(UserListContainer);
```

mapStateToProps函数的目的是使得连接的组件能轻松接收并渲染传递给它的数据，通常不是在组件中从redux store操作或取得数据。

- 选择器

  纯函数，接收store中的状态并计算数据，这些数据作为属性传递个React。

  选择器的主要作用是保持store中的状态数据最小，通过选择器来计算派生数据，例如过滤后的任务列表

  - reselect库 

    支持记忆和组合

## DvaJS

通过 reducers, effects 和 subscriptions 组织 model，简化 redux 和 redux-saga 引入的概念
-  react 表示法
   如果多个 Component 之间要发生交互, 那么状态(即: 数据)就维护在这些 Component 的最小公约父节点上, 子组件本身不维持任何 state, 完全由父组件传入 props 以决定其展现, 是一个纯函数的存在形式, 即: Pure Component
   
- redux 表示法
    React 只负责页面渲染, 而不负责页面逻辑, 页面逻辑可以从中单独抽取出来, 变成 store。
    Redux负责state状态机及页面逻辑
    - 状态有Store维护， 页面逻辑由reducer负责
    - 子组件多是pure component， 通过connect与store里的状态联系起来
    - 可以通过 dispatch 向 store 注入 action, 促使 store 的状态进行变化, 同时又订阅了 store 的状态变化, 一旦状态有变, 被 connect 的组件也随之刷新
    - 使用 dispatch 往 store 发送 action 的这个过程是可以被拦截的, 自然而然地就可以在这里增加各种 Middleware, 实现各种自定义功能, eg: logging
    - redux-saga 异步网络请求中间件

- dva 表示法
Dva 是基于 React + Redux + Saga 的最佳实践沉淀
    - 把 store 及 saga 统一为一个 model 的概念, 写在一个 js 文件里面
    - 增加了一个 Subscriptions, 用于收集其他来源的 action, eg: 键盘操作
    -  model 写法很简约, 类似于 DSL 或者 RoR, coding 快得飞起


### 样例
- Route
    页面代码
    ``` java
    
    ```
```

-  Module
    模块
-  Service
    网络请求
-  
```





