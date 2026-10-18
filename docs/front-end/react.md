---
title: React
tags:
  - 前端
  - React
  - JavaScript
  - Hooks
  - TypeScript
---

## 介绍 / Introduction

React 是 Meta 推出的**声明式、组件化** JavaScript 库，用于构建用户界面。你只需描述「UI 应该长什么样」，React 负责在数据变化时高效地更新 DOM。

!!! note
    本页基于 React 18/19 与函数组件 + Hooks 编写，覆盖从 JSX、状态、Hooks 到路由、状态管理、性能优化的完整速查。

## 核心思想 / Core Concept

| 思想 | 含义 | 实践体现 |
| --- | --- | --- |
| 声明式 | 描述结果而非过程 | 写 `UI = f(state)`，不手动操作 DOM |
| 组件化 | 页面拆分为可复用单元 | 每个组件 = 结构 + 逻辑 + 样式 |
| 单向数据流 | 数据从父流向子 | props 只读，状态变更靠父级回调 |
| 虚拟 DOM | 内存中的 UI 描述树 | diff 后只更新必要的真实 DOM |
| 状态驱动 | 状态变化触发重渲染 | 改 state 而非改 DOM |

```jsx
// 声明式：描述「点击按钮后标题变为 count」
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>Clicked {count}</button>;
}
```

## 创建项目 / Create Project

```bash
# Vite + React（推荐，快）
npm create vite@latest my-react-app -- --template react
npm create vite@latest my-react-app -- --template react-ts   # TS 模板

cd my-react-app
npm install
npm run dev

# 传统方式
npx create-react-app my-app
```

### 目录结构

```text
my-react-app/
├── public/                 # 静态资源，原样拷贝
├── src/
│   ├── assets/             # 需打包的图片/字体
│   ├── components/         # 通用组件
│   ├── pages/              # 页面级组件
│   ├── hooks/              # 自定义 Hook
│   ├── utils/              # 工具函数
│   ├── App.jsx             # 根组件
│   └── main.jsx            # 入口，挂载 ReactDOM
├── index.html              # 单页面宿主
├── package.json
└── vite.config.js
```

```jsx
// main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

// React 18+ 使用 createRoot 替代 ReactDOM.render
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

!!! tip
    `StrictMode` 会在开发环境**双调用** useEffect、渲染函数，用来暴露副作用问题；生产环境不生效，不要误以为是 bug。

## JSX 语法 / JSX

### 表达式与条件渲染

```jsx
const name = 'Tom';

function Profile({ user, isAdmin }) {
  return (
    <div>
      {/* 1. 花括号内是 JS 表达式 */}
      <h1>Hello, {name.toUpperCase()}</h1>

      {/* 2. 三元表达式 */}
      <p>{user.age >= 18 ? '成年' : '未成年'}</p>

      {/* 3. 逻辑与：仅当条件为真才渲染（注意左侧不要是 0） */}
      {isAdmin && <span>管理员</span>}

      {/* 4. 提前返回，处理空状态 */}
      {!user && <p>未登录</p>}
    </div>
  );
}
```

### 列表渲染与 key

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {/* key 必须唯一且稳定，优先用业务 id，避免用数组下标 */}
      {todos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

!!! warning
    用数组下标做 key 在「列表增删/排序」时会引发错误复用，优先使用业务唯一 id。列表为空时返回 `null` 而非空字符串占位。

### 样式：className / style / CSS Modules

```jsx
import './App.css';                       // 全局 CSS
import styles from './Button.module.css'; // CSS Modules

function App() {
  return (
    <>
      {/* class 是关键字，JSX 中用 className */}
      <div className="card">全局样式</div>
      {/* style 接收对象，属性名用驼峰 */}
      <div style={{ color: 'red', fontSize: 20 }}>行内样式</div>
      {/* CSS Modules：类名被局部作用域化 */}
      <button className={styles.btn}>模块化样式</button>
    </>
  );
}
```

```css
/* Button.module.css */
.btn {
  padding: 8px 16px;
  background: #1677ff;
  color: #fff;
  border-radius: 6px;
}
```

## 组件与 Props / Components & Props

```jsx
// 函数组件 + props 解构
function Card({ title, children, onClick = () => {} }) {
  return (
    <div className="card" onClick={onClick}>
      <h3>{title}</h3>
      {children}   {/* children 是标签包裹的内容 */}
    </div>
  );
}

// 使用
<Card title="标题" onClick={handleClick}>
  <p>这里是 children 内容</p>
</Card>
```

```jsx
// JS 项目：用 PropTypes 做运行时校验
import PropTypes from 'prop-types';

Card.propTypes = {
  title: PropTypes.string.isRequired,
  onClick: PropTypes.func,
};
```

```tsx
// TS 项目：用类型定义 props
interface CardProps {
  title: string;
  children?: React.ReactNode;
  onClick?: () => void;
}

function Card({ title, children, onClick }: CardProps) {
  return <div onClick={onClick}>{title}{children}</div>;
}
```

!!! tip
    props 是**只读**的，子组件不允许直接修改。默认值用解构默认参数（JS/TS 均可），比 `defaultProps` 更直观。

## State 与事件 / State & Events

### useState

```jsx
import { useState } from 'react';

function Form() {
  // 对象状态：更新时要新建对象，不可直接改原引用
  const [form, setForm] = useState({ name: '', age: 0 });
  const [list, setList] = useState([]);

  const update = (k, v) => setForm((prev) => ({ ...prev, [k]: v }));
  const add = (item) => setList((prev) => [...prev, item]);

  return <input value={form.name} onChange={(e) => update('name', e.target.value)} />;
}
```

```jsx
// 函数式更新：基于上一次状态计算，避免闭包拿到旧值
setCount((c) => c + 1);
setCount((c) => c + 1);   // 批处理下正确累加 2
```

!!! warning
    `setState` 是**异步批处理**的：同一事件内多次 set 不会立刻反映到本轮的 `count` 变量。需要用到最新值请用**函数式更新**。

### 事件处理与合成事件

```jsx
function Btn() {
  // React 使用合成事件，跨浏览器行为一致
  const handleClick = (e) => {
    e.preventDefault();       // 阻止默认行为
    e.stopPropagation();      // 阻止冒泡
    console.log(e.target);    // 触发元素
  };

  // 传参：用箭头函数包裹
  return <button onClick={(e) => handleClick(e)}>点我</button>;
}
```

### 受控 vs 非受控组件

| 维度 | 受控组件 | 非受控组件 |
| --- | --- | --- |
| 值来源 | React state | DOM 自身 |
| 读写方式 | `value` + `onChange` | `defaultValue` / `ref` |
| 实时校验 | 容易 | 较难 |
| 适用 | 表单大部分场景 | 文件上传、集成第三方 |

```jsx
// 受控
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />;

// 非受控：用 ref 读值
const inputRef = useRef(null);
<input ref={inputRef} defaultValue="hi" />;
// 读取：inputRef.current.value
```

## Hooks 全解 / Hooks

### useEffect — 副作用

```jsx
import { useEffect } from 'react';

useEffect(() => {
  // 1. 无依赖数组：每次渲染后都执行
});

useEffect(() => {
  // 2. 空数组：仅在挂载后执行一次（StrictMode 开发下执行两次）
  fetchData();
}, []);

useEffect(() => {
  // 3. 有依赖：依赖变化时执行
  console.log('count changed', count);
}, [count]);

useEffect(() => {
  const timer = setInterval(() => console.log('tick'), 1000);
  // 4. 清理函数：依赖变化/组件卸载前执行
  return () => clearInterval(timer);
}, []);
```

| 执行时机 | 依赖数组 | 说明 |
| --- | --- | --- |
| 每次渲染后 | 省略 | 慎用，易造成死循环 |
| 仅挂载后 | `[]` | 初始化请求、订阅 |
| 依赖变化后 | `[a, b]` | 最常见的用法 |
| 卸载/依赖变更前 | 返回函数 | 清理定时器、取消订阅、取消请求 |

!!! warning
    依赖数组必须包含 effect 内用到的**所有响应式值**（props/state/函数）。漏写会导致闭包读到旧值（stale closure）。

### useRef — 引用与持久化

```jsx
function Timer() {
  const domRef = useRef(null);          // 存 DOM 引用
  const countRef = useRef(0);           // 存可变值，改它不触发重渲染

  useEffect(() => {
    domRef.current.focus();
    countRef.current += 1;
  }, []);

  return <input ref={domRef} />;
}
```

!!! tip
    `useRef` 的值在渲染间**保持不变**且修改不触发重渲染，适合存定时器 id、上一次的值、DOM 节点。

### useMemo / useCallback — 缓存

```jsx
import { useMemo, useCallback } from 'react';

// useMemo：缓存「计算结果」
const sorted = useMemo(() => list.slice().sort((a, b) => a - b), [list]);

// useCallback：缓存「函数引用」
const handleClick = useCallback(() => setCount((c) => c + 1), []);
```

### useContext — 跨层级取值

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

// 顶层提供
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

// 深层消费
function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div>{theme}</div>;
}
```

### useReducer — 复杂状态

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'inc':
      return { count: state.count + action.step };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <>
      <button onClick={() => dispatch({ type: 'inc', step: 1 })}>+1</button>
      <button onClick={() => dispatch({ type: 'reset' })}>重置</button>
      <p>{state.count}</p>
    </>
  );
}
```

### Hook 选型对比

| Hook | 解决的问题 | 何时用 |
| --- | --- | --- |
| `useState` | 组件内部状态 | 简单、独立的可变数据 |
| `useReducer` | 多字段/多动作的状态 | 状态逻辑复杂、下一个状态依赖上一个 |
| `useEffect` | 副作用同步 | 请求、订阅、DOM 操作、定时器 |
| `useRef` | 跨渲染持久引用 | DOM 节点、定时器、非渲染值 |
| `useMemo` | 缓存计算结果 | 昂贵计算、稳定对象引用 |
| `useCallback` | 缓存函数引用 | 传给 memo 子组件/作为依赖 |
| `useContext` | 跨层级共享 | 主题、语言、登录用户 |

!!! note
    React 19 新增 `use()`（读取 Promise/Context）与 `useOptimistic`（乐观更新）等，可关注但非必需。

## 自定义 Hook / Custom Hooks

```jsx
// hooks/useFetch.js —— 封装请求
import { useState, useEffect } from 'react';

export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();  // 支持取消
    setLoading(true);
    fetch(url, { signal: controller.signal })
      .then((res) => res.json())
      .then(setData)
      .catch((err) => err.name !== 'AbortError' && setError(err))
      .finally(() => setLoading(false));

    return () => controller.abort();           // 卸载时取消请求
  }, [url]);

  return { data, loading, error };
}
```

```jsx
// hooks/useLocalStorage.js —— 持久化状态
import { useState, useEffect } from 'react';

export function useLocalStorage(key, initial) {
  const [value, setValue] = useState(() => {
    const raw = localStorage.getItem(key);
    return raw ? JSON.parse(raw) : initial;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// 使用
const [token, setToken] = useLocalStorage('token', '');
```

!!! tip
    自定义 Hook 就是「以 `use` 开头的函数」，内部可调用其他 Hook。命名必须以 `use` 开头，否则违反规则检查。

## 组件通信 / Component Communication

```jsx
// 1. 父传子：props
<Child title="hello" />

// 2. 子传父：父把回调传下去，子调用它
function Child({ onSend }) {
  return <button onClick={() => onSend('数据')}>发送</button>;
}
function Parent() {
  const [msg, setMsg] = useState('');
  return <><Child onSend={setMsg} /><p>{msg}</p></>;
}

// 3. 跨层级：Context（见 useContext 章节）
// 4. 兄弟组件：状态提升到共同父级，或用全局状态库
```

## 路由 / React Router

```bash
npm install react-router-dom
```

```jsx
// v6/v7：Routes + Route，元素用 element 而非 component
import { BrowserRouter, Routes, Route, Link, Outlet } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users/:id" element={<UserDetail />} />   {/* 动态参数 */}
        <Route path="/about" element={<About />}>
          <Route index element={<Intro />} />                  {/* 默认子路由 */}
          <Route path="team" element={<Team />} />             {/* 嵌套路由 */}
        </Route>
        <Route path="*" element={<NotFound />} />              {/* 兜底 */}
      </Routes>
    </BrowserRouter>
  );
}
```

```jsx
import { useParams, useNavigate, NavLink } from 'react-router-dom';

function UserDetail() {
  const { id } = useParams();          // 读取 /users/:id
  const navigate = useNavigate();      // 编程式跳转

  return (
    <>
      <h1>用户 {id}</h1>
      <NavLink to="/" className={({ isActive }) => (isActive ? 'active' : '')}>
        返回
      </NavLink>
      <button onClick={() => navigate('/about', { replace: true })}>去关于</button>
      <button onClick={() => navigate(-1)}>后退</button>
    </>
  );
}

// 嵌套路由的占位：子路由渲染在这里
function About() {
  return <><h2>About</h2><Outlet /></>;
}
```

```jsx
// 懒加载：按路由分割代码
import { lazy, Suspense } from 'react';
const Dashboard = lazy(() => import('./pages/Dashboard'));

<Route
  path="/dashboard"
  element={
    <Suspense fallback={<p>加载中...</p>}>
      <Dashboard />
    </Suspense>
  }
/>;
```

## 状态管理 / State Management

### 方案对比

| 方案 | 体积 | 学习成本 | 适用场景 |
| --- | --- | --- | --- |
| Context + useReducer | 内置 | 低 | 中小项目、主题/登录等少量全局状态 |
| Redux Toolkit | 中 | 中 | 大型团队、复杂状态流、需要 DevTools |
| Zustand | 很小 | 低 | 大多数项目，简洁且高性能 |

### 方案一：Context + useReducer

```jsx
const CountContext = createContext(null);

function countReducer(state, action) {
  switch (action.type) {
    case 'inc': return state + 1;
    case 'dec': return state - 1;
    default: return state;
  }
}

export function CountProvider({ children }) {
  const [count, dispatch] = useReducer(countReducer, 0);
  const value = useMemo(() => ({ count, dispatch }), [count]);
  return <CountContext.Provider value={value}>{children}</CountContext.Provider>;
}

export const useCount = () => useContext(CountContext);
```

!!! warning
    Context 值变化会让**所有消费者**重渲染。用 `useMemo` 稳定 value 中的对象引用，避免无谓刷新。

### 方案二：Redux Toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

```jsx
// store/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },      // Immer 允许「直接改」
    incrementBy: (state, action) => { state.value += action.payload; },
  },
});

export const { increment, incrementBy } = counterSlice.actions;
export default counterSlice.reducer;
```

```jsx
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({ reducer: { counter: counterReducer } });
```

```jsx
// 组件中使用
import { Provider, useSelector, useDispatch } from 'react-redux';
import { increment } from './store/counterSlice';

function Counter() {
  const value = useSelector((s) => s.counter.value);
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(increment())}>{value}</button>;
}

// 根组件注入 store
<Provider store={store}><Counter /></Provider>;
```

### 方案三：Zustand

```bash
npm install zustand
```

```jsx
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  inc: () => set((s) => ({ count: s.count + 1 })),
  reset: () => set({ count: 0 }),
}));

// 组件内按需选择，只订阅用到的切片，减少重渲染
function Counter() {
  const count = useStore((s) => s.count);
  const inc = useStore((s) => s.inc);
  return <button onClick={inc}>{count}</button>;
}
```

## 数据请求 / Data Fetching

```jsx
// 原生 fetch
useEffect(() => {
  fetch('/api/users')
    .then((res) => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json();
    })
    .then(setUsers)
    .catch(console.error);
}, []);
```

```jsx
// axios 封装（拦截器统一处理 token / 错误）
import axios from 'axios';

const request = axios.create({
  baseURL: '/api',
  timeout: 10000,
});

request.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

request.interceptors.response.use(
  (res) => res.data,
  (err) => {
    console.error(err.response?.data?.message || err.message);
    return Promise.reject(err);
  }
);

export default request;
```

!!! tip
    React 19 推荐用 `use()` 读取 Promise，或直接上 [TanStack Query](https://tanstack.com/query)：它内置缓存、重试、失效、乐观更新，省去手写 loading/error 状态。

```tsx
// TanStack Query 示例
import { useQuery } from '@tanstack/react-query';

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((r) => r.json()),
  });
  if (isLoading) return <p>加载中...</p>;
  if (error) return <p>出错了</p>;
  return <ul>{data.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

## 性能优化 / Performance

| 手段 | 作用 | 用法 |
| --- | --- | --- |
| `memo` | 缓存组件，props 不变则跳过渲染 | `export default memo(Comp)` |
| `useMemo` | 缓存计算结果 | 昂贵计算、稳定引用 |
| `useCallback` | 缓存函数引用 | 配合 memo 子组件 |
| `React.lazy` + `Suspense` | 代码分割懒加载 | 路由级/大组件 |
| 列表虚拟化 | 只渲染可视区域 | `react-window` / `react-virtualized` |
| 合理 key | 减少 diff 误差 | 用业务 id |

```jsx
import { memo, useMemo, useCallback, lazy, Suspense } from 'react';

// 只有 name 变化才重渲染
const UserItem = memo(function UserItem({ name, onClick }) {
  return <li onClick={onClick}>{name}</li>;
});

function List({ list }) {
  const total = useMemo(() => list.length, [list]);           // 缓存计算
  const handle = useCallback((id) => console.log(id), []);    // 稳定引用
  return <ul>{list.map((u) => <UserItem key={u.id} name={u.name} onClick={() => handle(u.id)} />)}</ul>;
}

// 懒加载
const HeavyChart = lazy(() => import('./HeavyChart'));
<Suspense fallback={<p>加载图表...</p>}><HeavyChart /></Suspense>;
```

!!! warning
    不要盲目加 `memo`/`useMemo`。只有**已验证的性能瓶颈**才优化，否则增加复杂度却收益甚微（React 官方建议先测量）。

## 常见坑 / Pitfalls

| 问题 | 现象 | 解决 |
| --- | --- | --- |
| 闭包陷阱 | 定时器/回调里拿到旧 state | 用函数式更新或把值放进依赖数组 |
| 依赖数组缺失 | effect 读到旧值、无限循环 | 补齐所有用到的响应式值 |
| setState 异步 | 循环内 set 只生效一次 | 用 `(prev) => ...` 函数式更新 |
| 直接改 state | UI 不更新 | 新建对象/数组（不可变更新） |
| key 用下标 | 增删后错位 | 用业务唯一 id |
| 条件为 0 渲染出 0 | `{list.length && <X/>}` 显示 0 | 写成 `{list.length > 0 && <X/>}` |
| effect 里 setState 无依赖 | 无限渲染 | 检查依赖，或加条件判断 |
| 卸载后 setState | 警告/内存泄漏 | 清理函数里 cancel/abort |

```jsx
// 闭包陷阱示例
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      // 错误：这里永远读到初始的 count = 0
      // setCount(count + 1);

      // 正确：函数式更新
      setCount((c) => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);   // 空依赖 + 函数式更新即可
}
```

!!! tip
    - 组件保持小而纯，把数据请求/副作用抽到自定义 Hook。
    - 状态能就近放就不上全局；全局状态优先考虑 Zustand。
    - 用 `key` 强制重建组件（如切换用户时 `<Form key={userId} />`）可重置内部状态。
