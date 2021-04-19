---
title: 关于 Redux 的一些思考
date: 2021-04-15 09:20:01
categories:
  - 框架
---

### Redux 出现的动机？

根据 Redux 官网给出的动机，总结如下

- 大型的应用状态多而复杂，导致你可能都不知道什么时候、什么原因、什么方式、导致了状态的变更
- 数据状态的可变性和异步，两者同时出现，导致管理数据很混乱

那 Redux 中是如何解决这些问题的呢，Redux 尝试通过更新的方式和时间加一定的限制，来让状态变化 变得可预测，在 Redux 中有单个原则：

1. 单一数据源，状态统一存储在一个 `store` 中
2. 状态是只读的，更改状态的唯一方式就是发出一个 `action` ，一个描述了发生了什么的对象
3. 使用存函数进行更改，也就是说 `reducer` 是一个存函数

### React Hooks 替代 Redux ？

#### 使用 Hooks 实现 简易 Redux

我们可以使用`createContext`、`useContext`，来存储全局状态，以及使用全局状态

先构造一个类似`Redux`中的`Provider`一样的组件用于包裹在 App 的最外层，使得子组件都可以通过`useContext`来获取全局状态

```jsx
// ./Provider.js
import { createContext, useReducer } from "react";

export const GlobalDataContext = createContext({
  count: 1,
});

const reducer = (state, action) => {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const initialStete = { count: 0 };

export function Provider(props) {
  const [data, dispatch] = useReducer(reducer, initialStete);
  return (
    <GlobalDataContext.Provider value={{ data, dispatch }}>
      {props.children}
    </GlobalDataContext.Provider>
  );
}

// ./index.js
ReactDOM.render(
  <React.StrictMode>
    <Data>
      <App />
    </Data>
  </React.StrictMode>,
  document.getElementById("root")
);
```

在子组件中使用`useContext`

```jsx
// ./App.js
import React, { useContext } from "react";
import { GlobalDataContext } from "./Provider";

export default function App() {
  const { data, dispatch } = useContext(GlobalData);
  return (
    <div>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <span>{data.count}</span>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </div>
  );
}
```

#### Redux 的优势

以上我们就使用`hooks`完成了一个简易的`Redux`, 但是 hooks 真的能替代 Redux 吗？ 显然我觉得是不能的

- Redux 有成熟的数据模块化机制，我们可以创建多个 Reducer，使用 `combineReducer` 来合并并存储在一个 store 中
- Redux 有成熟的中间件机制
- Redux 有成熟的调试工具

Hooks 确实能达到 Redux 的一些功能特性，但是相比 Redux 还是有一定的区别的，当然我们可能使用 hooks 来封装一个 Redux，但是有现成的优秀的轮子了，何必又再造轮子呢？

### 是否需要 Immutable？

Redux 中当数据发生改表需要你返回一个新的对象，也就是数据不可变的方式更改状态，这有助于 React 中的性能优化，我们不一定需要 Immer 或者 Immutable.js 这样的不可变库来返回一个新的对象，我们可以使用 concat slice 或者 Object.assign 拓展运算符等方式来返回一个新的对象引用，但是当数据比较复杂的情况下使用 Immutable 的数据结构是比较高效的

### Redux vs MobX？

- MobX 没有模板代码，相对简洁，而对于 Redux 需要创建 store action reducer 的模板代码，相对繁琐
- 对于异步任务，Redux 还需要引用`Redux-thunk`等工具，而 Mobx 不需要
- 数据的不可变性, 在 MobX 是可以直接使用 JS 进行赋值修改的，而 Redux 中的数据是只读，若更新数据则要返回新的对象
- MobX 过于自由，容易导致团队代码风格不统一，可维护性相对 Redux 差些
- Redux 的 toolkit 帮我解决了以上 redux 的不足

### `redux-thunk` 做了什么 ?

redux-thunk 使得 dispatch 能接收一个函数，这样就能解决，我们再 dispatch 的时候能发出多个动作，比如发送请求前，和收到数据的时候各发送一个 action

### Redux 中的 compse

```javascript
// 高阶函数，将函数从右到做的执行
function compose(...funcs) {
  if (funcs.length === 0) {
    (args) => args;
  }
  if (funcs.length === 1) {
    return funcs[0];
  }
  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
```

###
