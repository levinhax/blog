# useState

useState()用于为函数组件引入状态（state）。纯函数不能有状态，所以把状态放在钩子里面。

```
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [user, setUser] = useState({ name: "leon", age: 18 });

  return (
    <div className="App">
      <p>hello {user.name}, You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}

export default App;
```

#### 不可局部更新

如果state是一个对象，能否部分setState？
答案是不行，因为setState, useReducer都不会帮我们合并属性

#### 地址要变

setState(obj) 如果obj地址不变，那么React就认为数据没有变化，不会更新视图

#### useState接收函数

```
const [state, setState] = useState(() => {return initialState})

// 该函数返回初始state，且只执行一次

const [user, setUser] = useState(() => ({ name: "leon", age: 18 }));
```

#### setState接收函数

```
setN(i => i + 1)

// 如果你能接收这种形式，应该优先使用这种形式
```

```
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(i => i + 1)
  }

  return (
    <div className="App">
      <p>You clicked {count} times</p>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}

export default App;
```

# useEffect

对环境的改变即为副作用，如修改 document.title，但我们不一定非要把副作用放在 useEffect 里面，实际上叫做 afterRender 更好，每次render后执行

用法：

1. 作为 componentDidMount 使用，[ ] 作第二个参数
2. 作为 componentDidUpdate 使用，可指定依赖
3. 作为 componentWillUnmount 使用，通过 return
4. 以上三种用途可同时存在

如果同时存在多个 useEffect， 会按照出现次序执行

```
import React, { useState, useEffect } from "react";

function App() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(i => i + 1);
  };

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]);

  return (
    <div className="App">
      <p>You clicked {count} times</p>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}

export default App;


// =================================================
// 第一次渲染时执行（componentDidMount）
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, []);

// 依赖变化时执行（componentDidUpdate）
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]);

// 任何一个state变化都会执行
useEffect(() => {
  document.title = `You clicked ${count} times`;
});

// 组件卸载的时候执行清除操作（componentWillUnmount）
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
});
```

# useContext

接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。

调用了 useContext 的组件总会在 context 值变化时重新渲染。如果重渲染组件的开销较大，你可以 通过使用 memoization 来优化。

- 使用 C = createContext(initial) 创建上下文
- 使用 C.Provider 圈定作用域
- 在作用域内使用 useContext(C)来使用上下文

```
import React, { useContext } from "react";

const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      {" "}
      I am styled by theme context!{" "}
    </button>
  );
}

export default App;
```

# useReducer

```
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。

```
// Counter.js

import React, { useReducer } from "react";

function init(initialCount) {
  return { count: initialCount };
}
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({ type: "reset", payload: initialCount })}
      >
        {" "}
        Reset
      </button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}

export default Counter;


// App.js
import React from "react";
import Counter from "./views/Counter";

function App() {
  return (
    <>
      <Counter initialCount={10} />
    </>
  );
}

export default App;
```

如果 Reducer Hook 的返回值与当前 state 相同，React 将跳过子组件的渲染及副作用的执行。（React 使用 Object.is 比较算法 来比较 state。）

需要注意的是，React 可能仍需要在跳过渲染前再次渲染该组件。不过由于 React 不会对组件树的"深层"节点进行不必要的渲染，所以大可不必担心。如果你在渲染期间执行了高开销的计算，则可以使用 useMemo 来进行优化。

# useCallback

返回一个 memoized 回调函数
```
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

useCallback(fn, deps) 相当于 useMemo(() => fn, deps)。

# useMemo

```
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

把"创建"函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

记住，传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。

如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。

# useRef

```
const refContainer = useRef(initialValue);
```

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

一个常见的用例便是命令式地访问子组件：
```
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

useRef() 比 ref 属性更有用。它可以很方便地保存任何可变值，其类似于在 class 中使用实例字段的方式。因为它创建的是一个普通 Javascript 对象。而 useRef() 和自建一个 {current: ...} 对象的唯一区别是，useRef 会在每次渲染时返回同一个 ref 对象。

# useLayoutEffect

其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

尽可能使用标准的 useEffect 以避免阻塞视觉更新。
