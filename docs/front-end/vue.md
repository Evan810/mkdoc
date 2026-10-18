---
title: Vue
tags:
  - 前端
  - Vue
  - JavaScript
  - TypeScript
  - Vue 3
  - 组合式 API
---

## 介绍 / Introduction

Vue.js 是一套用于构建用户界面的渐进式 JavaScript 框架，以**响应式数据绑定**与**组件化**为核心特性。Vue 3 在保留模板语法的易用性的同时，引入了**组合式 API（Composition API）**，让逻辑组织更灵活、类型推导更友好。

!!! note
    本文示例统一基于 Vue 3 + `<script setup>` + TypeScript 写法，目标版本为 Vue 3.4+。

## 核心概念与 Vue 2 对比 / Core Concepts

### 选项式 vs 组合式

| 维度 | 选项式 API（Vue 2 风格） | 组合式 API（Vue 3 推荐） |
| --- | --- | --- |
| 代码组织 | 按 `data`/`methods`/`computed` 分块 | 按**逻辑关注点**聚合 |
| 复用方式 | `mixin`（来源不透明、易命名冲突） | 组合式函数（来源清晰、可传参） |
| 类型推导 | 对 TypeScript 不友好 | 天然友好，`ref` 可精确推导 |
| `this` | 依赖 `this`，箭头函数易踩坑 | 几乎不依赖 `this` |
| 打包体积 | 组件实例 options 合并开销 | 逻辑独立，tree-shaking 更彻底 |
| 适用场景 | 简单小组件 | 中大型项目、复杂逻辑复用 |

!!! tip
    两者**可以共存**，同一个项目里选项式组件与 `<script setup>` 组件互不影响，迁移可渐进进行。

### 关键变化速览

| 变化点 | Vue 2 | Vue 3 |
| --- | --- | --- |
| 响应式实现 | `Object.defineProperty` | `Proxy`（可监听数组索引、新增/删除属性） |
| 根实例 | `new Vue({...})` | `createApp({...})` |
| 多根节点 | 单根，需包裹 `<div>` | 支持**多根** Fragment |
| `data` | 必须是函数返回对象 | 同左（根组件也是） |
| 过滤器 `filters` | 支持 | **已移除**，改用 computed/方法 |
| `$set` / `$delete` | 需要 | **不再需要**（Proxy 自动追踪） |
| v-model 组件 | `value` + `input` | `modelValue` + `update:modelValue` |
| 事件总线 | `$emit` / `$on` | **移除**，改用 mitt 或 Pinia |
| 生命周期 | `beforeDestroy` 等 | `beforeUnmount` 等（见对照表） |

## 项目创建与工程结构 / Project Setup

### 脚手架创建

```bash
# 官方脚手架，交互式选择 TypeScript / Router / Pinia / ESLint 等
npm create vue@latest my-app

cd my-app
npm install
npm run dev        # 本地开发，默认 http://localhost:5173
npm run build      # 生产构建，产物在 dist/
npm run preview    # 本地预览构建产物
```

### 目录结构

```text
my-app/
├── index.html                  # 入口 HTML（挂载点 #app）
├── vite.config.ts              # Vite 配置（别名、代理、插件）
├── tsconfig.json               # TS 配置
├── package.json
└── src/
    ├── main.ts                 # 应用入口，createApp().mount()
    ├── App.vue                 # 根组件
    ├── assets/                 # 静态资源
    ├── components/             # 通用组件
    ├── views/                  # 路由页面
    ├── router/index.ts         # 路由配置
    ├── stores/                 # Pinia 状态仓库
    ├── composables/            # 组合式函数（自定义 Hook）
    └── utils/                  # 工具函数
```

### 应用入口 main.ts

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.mount('#app')          // 挂载到 index.html 中的 #app
```

!!! warning
    Vue 3 使用 `createApp` 创建**应用实例**，注册全局组件/指令/插件都应调用 `app.xxx`，而非 `Vue.xxx`。

## 模板语法 / Template Syntax

### 插值

```vue
<script setup lang="ts">
const msg = '你好，Vue'
const rawHtml = '<span style="color:red">富文本</span>'
</script>

<template>
  <!-- 文本插值，Mustache 语法 -->
  <p>{{ msg }}</p>
  <!-- 渲染 HTML，注意 XSS 风险，仅用于可信内容 -->
  <p v-html="rawHtml"></p>
</template>
```

### 常用指令速查

| 指令 | 作用 | 示例 |
| --- | --- | --- |
| `v-text` | 设置文本内容 | `<p v-text="msg"></p>` |
| `v-html` | 渲染 HTML 字符串 | `<p v-html="html"></p>` |
| `v-show` | 切换 `display`，元素始终渲染 | `<p v-show="ok"></p>` |
| `v-if` / `v-else-if` / `v-else` | 条件渲染，条件为假时不渲染 | `<p v-if="n > 0">正</p>` |
| `v-for` | 列表渲染 | `<li v-for="(i, idx) in list">{{ idx }}</li>` |
| `v-bind`（`:`) | 绑定属性 | `<img :src="url">` |
| `v-on`（`@`) | 绑定事件 | `<button @click="onClick">` |
| `v-model` | 双向绑定 | `<input v-model="text">` |
| `v-once` | 只渲染一次，跳过后续更新 | `<p v-once>{{ n }}</p>` |
| `v-memo` | 依赖不变时跳过子树更新 | `<div v-memo="[a]">…</div>` |

!!! tip
    `v-if` 与 `v-show` 的选择：**频繁切换**用 `v-show`（仅改 CSS），**初始条件很少为真**或需按需创建用 `v-if`。

### 条件渲染与列表

```vue
<script setup lang="ts">
import { ref } from 'vue'

const score = ref(85)
const list = ref([
  { id: 1, name: '苹果' },
  { id: 2, name: '香蕉' },
])
</script>

<template>
  <p v-if="score >= 90">优秀</p>
  <p v-else-if="score >= 60">及格</p>
  <p v-else>不及格</p>

  <!-- 必须给 :key，且用稳定唯一值，避免用 index -->
  <ul>
    <li v-for="item in list" :key="item.id">{{ item.name }}</li>
  </ul>
</template>
```

!!! warning
    不要在同一个元素上同时使用 `v-if` 与 `v-for`（Vue 3 中 `v-if` 优先级更高，会取不到 `v-for` 的变量）。应当用 `<template>` 包裹或提前用 computed 过滤。

### 动态参数

```vue
<script setup lang="ts">
const attrName = 'title'
const eventName = 'focus'
</script>

<template>
  <!-- 动态属性名 -->
  <a v-bind:[attrName]="'提示'">链接</a>
  <!-- 动态事件名 -->
  <input v-on:[eventName]="() => console.log('聚焦')">
</template>
```

### 事件修饰符

| 修饰符 | 作用 |
| --- | --- |
| `.stop` | 阻止事件冒泡，等价 `event.stopPropagation()` |
| `.prevent` | 阻止默认行为，等价 `event.preventDefault()` |
| `.self` | 仅当事件目标是元素自身时触发 |
| `.once` | 事件只触发一次 |
| `.capture` | 使用捕获模式 |
| `.passive` | 提升滚动性能，无需等待处理完成 |

```html
<!-- 阻止冒泡 + 阻止默认行为 -->
<a href="/" @click.stop.prevent="onClick">跳转</a>
<!-- 只触发一次 -->
<button @click.once="handleOnce">只点一次</button>
```

### 表单修饰符

| 修饰符 | 作用 |
| --- | --- |
| `.lazy` | 将 `input` 事件改为 `change` 后再同步 |
| `.number` | 自动转为数字类型 |
| `.trim` | 自动去除首尾空格 |

```html
<input v-model.lazy="text">            <!-- 失去焦点时才同步 -->
<input v-model.number="age" type="number">
<input v-model.trim="username">
```

## 响应式系统 / Reactivity

### ref 与 reactive

```typescript
import { ref, reactive } from 'vue'

// ref：适合基本类型，任意类型，需通过 .value 读写
const count = ref(0)
count.value++

// reactive：只能用于对象/数组/Map/Set，直接访问属性
const state = reactive({ name: '张三', age: 18 })
state.age++
```

| 维度 | `ref` | `reactive` |
| --- | --- | --- |
| 适用类型 | 任意类型（基本类型、对象） | 仅对象类型 |
| 访问方式 | 需 `.value`（模板中自动解包） | 直接 `.属性` |
| 重新赋值 | 可整体替换 `count.value = x` | 不可整体替换，会丢失响应式 |
| 推荐场景 | 大多数场景优先用 `ref` | 一组关联状态、复杂对象 |

!!! tip
    Vue 官方建议**优先使用 `ref`**：类型推导更清晰，且能避免 `reactive` 解构、整体替换丢失响应式的坑。

### computed 计算属性

```typescript
import { ref, computed } from 'vue'

const price = ref(100)
const qty = ref(3)

// 只读计算属性，有缓存，依赖不变则不重新计算
const total = computed(() => price.value * qty.value)

// 可写计算属性
const doublePrice = computed({
  get: () => price.value * 2,
  set: (val: number) => { price.value = val / 2 },
})
```

### watch 与 watchEffect

```typescript
import { ref, watch, watchEffect } from 'vue'

const keyword = ref('')
const user = ref({ name: '张三', age: 18 })

// 监听单个 ref
watch(keyword, (newVal, oldVal) => {
  console.log(`从 ${oldVal} 变为 ${newVal}`)
})

// 监听多个源
watch([keyword, () => user.value.age], ([kw, age]) => {
  console.log(kw, age)
})

// 深度监听对象（性能开销较大），immediate 立即执行
watch(user, (val) => console.log(val), { deep: true, immediate: true })

// watchEffect：自动收集依赖，立即执行一次，无需指定源
const stop = watchEffect(() => {
  console.log('依赖变化即执行', keyword.value)
})
// stop() 可手动停止
```

| 对比项 | `watch` | `watchEffect` |
| --- | --- | --- |
| 依赖收集 | 需显式指定监听源 | 自动收集回调中用到的响应式数据 |
| 是否立即执行 | 默认否，可 `immediate: true` | **默认立即执行** |
| 获取旧值 | 可拿到 `oldVal` | 拿不到旧值 |
| 适用场景 | 需要旧值、精确控制 | 副作用连带多个依赖 |

### 解构导致的响应式丢失

```typescript
import { reactive, toRefs } from 'vue'
import { storeToRefs } from 'pinia'

const state = reactive({ name: '张三', age: 18 })

// ❌ 错误：解构后 name 变成普通值，失去响应式
const { name } = state

// ✅ 正确：用 toRefs 把每个属性转成 ref
const { name: nameRef, age } = toRefs(state)

// ✅ Pinia 仓库解构同样用 storeToRefs，避免失去响应式
const userStore = useUserStore()
const { count } = storeToRefs(userStore)   // state/getters 保持响应式
const { increment } = userStore            // actions 直接解构即可
```

### nextTick 等待 DOM 更新

```typescript
import { ref, nextTick } from 'vue'

const list = ref<string[]>([])
const inputRef = ref<HTMLInputElement | null>(null)

async function add() {
  list.value.push('新项')
  // 数据已变但 DOM 还未更新，等下一次 DOM 更新周期后再操作
  await nextTick()
  console.log(list.value.length)             // 读到最新 DOM 数量
  inputRef.value?.focus()
}
```

## 生命周期钩子 / Lifecycle Hooks

### Vue 2 与 Vue 3 对照表

| Vue 2 选项式 | Vue 3 组合式 | 说明 |
| --- | --- | --- |
| `beforeCreate` | 用 `setup()` 本身替代 | 实例初始化前 |
| `created` | 用 `setup()` 本身替代 | 实例创建完成，可访问响应式数据 |
| `beforeMount` | `onBeforeMount` | 挂载前 |
| `mounted` | `onMounted` | 挂载完成，可访问 DOM |
| `beforeUpdate` | `onBeforeUpdate` | 数据更新、DOM 未更新前 |
| `updated` | `onUpdated` | DOM 更新后 |
| `beforeDestroy` | `onBeforeUnmount` | 卸载前（**改名**） |
| `destroyed` | `onUnmounted` | 卸载后（**改名**） |
| `errorCaptured` | `onErrorCaptured` | 捕获后代组件错误 |
| `activated` / `deactivated` | `onActivated` / `onDeactivated` | keep-alive 缓存组件激活/停用 |

```vue
<script setup lang="ts">
import { onMounted, onUnmounted, onUpdated } from 'vue'

onMounted(() => {
  console.log('组件挂载，可访问 DOM')
  window.addEventListener('resize', onResize)
})

onUpdated(() => console.log('视图已更新'))

onUnmounted(() => {
  // 清理副作用：定时器、事件监听、WebSocket 等
  window.removeEventListener('resize', onResize)
})
</script>
```

!!! warning
    `<script setup>` 中没有 `beforeCreate`/`created` 钩子，其逻辑**直接写在 setup 顶层**即可，此时组件尚未挂载，拿不到 DOM。

## 组件 / Components

### defineProps / defineEmits

```vue
<!-- Child.vue -->
<script setup lang="ts">
// TS 类型声明式 props（Vue 3.3+ 支持类型引入）
interface Props {
  title: string
  count?: number          // 可选
  list?: string[]
}
const props = withDefaults(defineProps<Props>(), {
  count: 0,
  list: () => [],         // 对象/数组默认值必须用工厂函数
})

// 定义事件，TS 泛型写法（Vue 3.3+）
const emit = defineEmits<{
  change: [value: number]     // 具名 + 参数类型
  submit: []
}>()

function handleClick() {
  emit('change', props.count + 1)
}
</script>

<template>
  <p>{{ title }}：{{ count }}</p>
  <button @click="handleClick">+1</button>
</template>
```

```vue
<!-- Parent.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import Child from './Child.vue'

const num = ref(0)
</script>

<template>
  <!-- kebab-case 传参，子组件里是 camelCase -->
  <Child title="计数器" :count="num" @change="num = $event" />
</template>
```

!!! note
    `defineProps` / `defineEmits` / `defineExpose` 是**编译器宏**，无需 import，只能在 `<script setup>` 顶层使用。

### defineExpose 暴露实例方法

```vue
<!-- Child.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
function reset() {
  count.value = 0
}

// 仅暴露需要给父组件访问的内容（默认组件实例完全封闭）
defineExpose({ reset })
</script>
```

```vue
<!-- Parent.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const childRef = ref<InstanceType<typeof Child> | null>(null)

onMounted(() => {
  childRef.value?.reset()          // 调用子组件暴露的方法
})
</script>

<template>
  <Child ref="childRef" />
</template>
```

### 插槽 / Slots

```vue
<!-- Layout.vue -->
<template>
  <header>
    <!-- 默认插槽，可提供后备内容 -->
    <slot>默认标题</slot>
  </header>
  <main>
    <!-- 具名插槽 -->
    <slot name="content"></slot>
  </main>
  <footer>
    <!-- 作用域插槽：把子组件数据传给父组件 -->
    <slot name="footer" :year="2026" :author="'张三'"></slot>
  </footer>
</template>
```

```vue
<!-- 使用方 -->
<template>
  <Layout>
    <!-- 默认插槽 -->
    <template #default>首页</template>
    <!-- 具名插槽，可 shorthand -->
    <template #content>
      <p>正文内容</p>
    </template>
    <!-- 作用域插槽，解构子组件传出的数据 -->
    <template #footer="{ year, author }">
      <p>© {{ year }} {{ author }}</p>
    </template>
  </Layout>
</template>
```

## 组件通信 / Component Communication

| 方式 | 方向 | 适用场景 |
| --- | --- | --- |
| `props` | 父 → 子 | 基础数据传递，单向数据流 |
| `emit` | 子 → 父 | 子组件通知父组件、回传数据 |
| `v-model` | 双向 | 表单类组件双向绑定 |
| `provide / inject` | 祖先 → 后代 | 跨层级传递，避免逐层透传 |
| `attrs`（`useAttrs`） | 父 → 子 | 透传未声明为 props 的属性/事件 |
| 模板 `ref` | 父 → 子 | 父组件直接调用子组件暴露的方法 |
| Pinia | 任意 | 全局状态共享，跨页面/跨组件 |
| mitt 事件总线 | 任意 | 极简跨组件事件（Vue 3 无内置总线） |

### provide / inject

```typescript
// 祖先组件
import { provide, ref } from 'vue'

const theme = ref<'light' | 'dark'>('light')
const toggleTheme = () => (theme.value = theme.value === 'light' ? 'dark' : 'light')

// 提供值 + 方法，key 建议用 Symbol 或 InjectionKey
provide('theme', { theme, toggleTheme })
```

```typescript
// 任意后代组件
import { inject } from 'vue'

// 第二个参数为默认值；使用 Symbol key 时需加类型断言
const { theme, toggleTheme } = inject('theme') as {
  theme: Ref<'light' | 'dark'>
  toggleTheme: () => void
}
```

### useAttrs 透传

```vue
<!-- MyButton.vue：不声明 class/自定义事件，自动透传到根元素 -->
<script setup lang="ts">
import { useAttrs } from 'vue'
const attrs = useAttrs()          // 非 props 的属性集合
</script>

<template>
  <button v-bind="attrs" class="my-btn">
    <slot></slot>
  </button>
</template>
```

!!! tip
    Vue 3.4+ 可用 `defineOptions` 或 `inheritAttrs: false` 控制属性透传；透传 class/style 会自动合并。

## 路由 / Vue Router 4

### 安装与基础配置

```bash
npm install vue-router@4
```

```typescript
// src/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  { path: '/', name: 'home', component: () => import('@/views/Home.vue') },
  { path: '/about', name: 'about', component: () => import('@/views/About.vue') },
  // 动态路由参数
  { path: '/user/:id', name: 'user', component: () => import('@/views/User.vue'), props: true },
  // 404 兜底（放在最后）
  { path: '/:pathMatch(.*)*', name: 'notFound', component: () => import('@/views/NotFound.vue') },
]

const router = createRouter({
  history: createWebHistory(),     // HTML5 模式，需服务器配置回退
  routes,
})

export default router
```

### 嵌套路由

```typescript
const routes: RouteRecordRaw[] = [
  {
    path: '/admin',
    component: () => import('@/views/AdminLayout.vue'),
    children: [
      // 默认子路由
      { path: '', name: 'admin-home', component: () => import('@/views/AdminHome.vue') },
      { path: 'users', name: 'admin-users', component: () => import('@/views/AdminUsers.vue') },
      { path: 'settings', name: 'admin-settings', component: () => import('@/views/AdminSettings.vue') },
    ],
  },
]
```

```vue
<!-- AdminLayout.vue 中使用 <router-view> 渲染子路由 -->
<template>
  <aside>侧边栏</aside>
  <main>
    <router-view></router-view>
  </main>
</template>
```

### 编程式导航与参数

```typescript
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

router.push('/home')                                   // 路径
router.push({ name: 'user', params: { id: '42' } })    // 命名路由 + params
router.push({ path: '/list', query: { page: '2' } })   // query
router.replace({ name: 'home' })                       // 替换历史，不可后退
router.back()                                          // 返回

// 读取参数
route.params.id      // 动态路由参数（String | String[]）
route.query.page     // 查询参数
```

| 方式 | 语法 | 取参 |
| --- | --- | --- |
| params | `router.push({ name, params })` | `route.params` |
| query | `router.push({ path, query })` | `route.query` |
| 声明式 | `<router-link :to="{ name: 'user', params }">` | 同左 |

### 导航守卫

```typescript
// 全局前置守卫：常用于鉴权
router.beforeEach((to, from) => {
  const token = localStorage.getItem('token')
  // 返回 false 取消导航；返回字符串/对象表示重定向
  if (to.meta.requiresAuth && !token) {
    return { name: 'login', query: { redirect: to.fullPath } }
  }
})

router.afterEach((to) => {
  document.title = (to.meta.title as string) || '默认标题'
})

// 路由独享守卫
{ path: '/admin', component: Admin, beforeEnter: (to) => { /* ... */ } }
```

```vue
<!-- 组件内守卫（组合式写法） -->
<script setup lang="ts">
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

// 离开当前路由时：常用于表单未保存提示
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChange.value) return window.confirm('确定离开？')
})

// 同一组件复用、参数变化时
onBeforeRouteUpdate((to) => console.log('参数变为', to.params.id))
</script>
```

!!! tip
    `route.meta` 可携带自定义元信息（如 `requiresAuth`、`title`）。类型安全可用 `declare module 'vue-router'` 扩展 `RouteMeta`。

## 状态管理 Pinia

### 安装与注册

```bash
npm install pinia
```

```typescript
// main.ts
import { createPinia } from 'pinia'
app.use(createPinia())
```

### 定义 Store（组合式写法，推荐）

```typescript
// src/stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // state
  const token = ref('')
  const profile = ref<{ name: string; age: number } | null>(null)

  // getters
  const isLogin = computed(() => !!token.value)

  // actions
  async function login(username: string) {
    // 模拟请求
    token.value = 'mock-token'
    profile.value = { name: username, age: 18 }
  }
  function logout() {
    token.value = ''
    profile.value = null
  }

  return { token, profile, isLogin, login, logout }
})
```

### 选项式写法（对比）

```typescript
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++        // 选项式中可用 this
    },
  },
})
```

### 在组件中使用

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
// 结构 state/getters 必须用 storeToRefs 保持响应式
const { token, isLogin } = storeToRefs(userStore)
// actions 直接解构
const { login, logout } = userStore
</script>

<template>
  <p v-if="isLogin">已登录：{{ token }}</p>
  <button @click="login('张三')">登录</button>
</template>
```

### 持久化

```bash
npm install pinia-plugin-persistedstate
```

```typescript
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)
app.use(pinia)
```

```typescript
// 在 store 中开启持久化（默认存 localStorage）
export const useUserStore = defineStore('user', () => { /* ... */ }, {
  persist: {
    key: 'user-store',
    storage: localStorage,
    pick: ['token'],           // 只持久化部分字段
  },
})
```

| 方案 | 存储位置 | 特点 |
| --- | --- | --- |
| `localStorage` | 浏览器本地 | 手动/插件持久化，刷新不丢，长期有效 |
| `sessionStorage` | 会话级 | 关闭标签页即失效 |
| 手动 watch | — | `watch(store.$state, ...)` 自己写入 storage |

## 表单处理 / Forms

### 基础 v-model

```vue
<script setup lang="ts">
import { ref, reactive } from 'vue'

const username = ref('')
const agree = ref(false)                 // 单个复选框 → boolean
const hobbies = ref<string[]>([])        // 多个复选框 → 数组
const gender = ref('male')               // 单选按钮 → 选中值
const city = ref('')                     // 下拉框

const form = reactive({ name: '', age: 18 })
</script>

<template>
  <input v-model="username" placeholder="用户名">
  <input type="checkbox" v-model="agree"> 同意协议
  <input type="checkbox" value="篮球" v-model="hobbies">
  <input type="checkbox" value="音乐" v-model="hobbies">
  <input type="radio" value="male" v-model="gender">
  <input type="radio" value="female" v-model="gender">
  <select v-model="city">
    <option value="">请选择</option>
    <option value="bj">北京</option>
  </select>
</template>
```

### 组件上的 v-model（单个）

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
// 默认 prop 名是 modelValue，默认事件是 update:modelValue
const model = defineModel<string>({ required: true })   // Vue 3.4+ 语法糖
</script>

<template>
  <input :value="model" @input="model = ($event.target as HTMLInputElement).value">
</template>
```

```vue
<!-- 父组件 -->
<template>
  <CustomInput v-model="username" />
</template>
```

### 多个 v-model

```vue
<!-- UserName.vue -->
<script setup lang="ts">
const firstName = defineModel<string>('firstName')
const lastName = defineModel<string>('lastName')
</script>

<template>
  <input v-model="firstName">
  <input v-model="lastName">
</template>
```

```html
<!-- 父组件：为每个 v-model 指定名字 -->
<UserName v-model:first-name="first" v-model:last-name="last" />
```

!!! tip
    Vue 3.4 之前需要手写 `props: ['modelValue']` + `emit('update:modelValue')`；3.4+ 推荐使用 `defineModel`。

## 样式 / Styles

### scoped 与深度选择器

```vue
<style scoped>
/* 只作用于当前组件元素，通过给元素加唯一属性实现 */
.title { color: #42b883; }

/* :deep() 穿透到子组件内部（如修改第三方组件样式） */
:deep(.el-input__inner) { border-radius: 8px; }

/* :slotted() 作用于插槽内容 */
:slotted(p) { margin: 0; }

/* :global() 全局生效 */
:global(.no-margin) { margin: 0; }
</style>
```

### CSS Modules

```vue
<template>
  <!-- 通过 $style 访问模块化类名 -->
  <p :class="$style.red">红色文本</p>
</template>

<style module>
.red { color: red; }
</style>
```

| 方案 | 类名处理 | 适用场景 |
| --- | --- | --- |
| `<style scoped>` | 加属性选择器隔离 | 大多数组件样式隔离 |
| CSS Modules | 编译为哈希类名 | 需要 JS 动态操作类名 |
| 全局 `<style>` | 不隔离 | 全局主题、重置样式 |

!!! warning
    `scoped` 只保证**当前组件元素**不互相污染，若子组件根元素继承父类名仍可能被穿透；复杂子树可用 `:deep()` 明确处理。

## 网络请求 / HTTP with Axios

### 安装与封装

```bash
npm install axios
```

```typescript
// src/utils/request.ts
import axios, { type AxiosInstance, type InternalAxiosRequestConfig } from 'axios'

const request: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,   // 来自 .env 文件
  timeout: 10000,
})

// 请求拦截器：统一携带 token
request.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = localStorage.getItem('token')
    if (token) config.headers.Authorization = `Bearer ${token}`
    return config
  },
  (error) => Promise.reject(error),
)

// 响应拦截器：统一处理业务码与错误
request.interceptors.response.use(
  (response) => {
    const res = response.data
    // 约定 code === 0 表示成功
    if (res.code !== 0) {
      if (res.code === 401) { /* 跳转登录 */ }
      return Promise.reject(new Error(res.message || '请求失败'))
    }
    return res.data
  },
  (error) => {
    // 网络错误 / 超时 / HTTP 状态码异常
    console.error('请求错误：', error.message)
    return Promise.reject(error)
  },
)

export default request
```

### 使用与泛型封装

```typescript
// src/api/user.ts
import request from '@/utils/request'

export interface UserInfo {
  id: number
  name: string
  age: number
}

// 响应拦截器已解包 data，此处泛型直接是业务数据类型
export const getUser = (id: number) =>
  request.get<UserInfo>(`/user/${id}`)

export const updateUser = (data: UserInfo) =>
  request.put<void>('/user', data)
```

```typescript
// 组件中调用
import { getUser } from '@/api/user'

const info = await getUser(1)
console.log(info.name)
```

!!! warning
    拦截器里解包 `response.data` 属于常见约定，但会改变 `request.get` 的返回类型语义，团队内需**统一约定**，避免混用导致类型错乱。

## 组合式函数 / Composables

组合式函数（自定义 Hook）是以 `use` 开头的复用逻辑函数，内部可使用 `ref`/`computed`/生命周期钩子。

```typescript
// src/composables/useMouse.ts —— 追踪鼠标位置
import { ref, onMounted, onUnmounted, type Ref } from 'vue'

export function useMouse(): { x: Ref<number>; y: Ref<number> } {
  const x = ref(0)
  const y = ref(0)

  function update(e: MouseEvent) {
    x.value = e.pageX
    y.value = e.pageY
  }

  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  return { x, y }
}
```

```typescript
// src/composables/useRequest.ts —— 通用请求封装
import { ref } from 'vue'

export function useRequest<T>(fetcher: () => Promise<T>) {
  const data = ref<T>()
  const loading = ref(false)
  const error = ref<Error>()

  async function run() {
    loading.value = true
    error.value = undefined
    try {
      data.value = await fetcher()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, run }
}
```

```vue
<script setup lang="ts">
import { useMouse } from '@/composables/useMouse'
import { useRequest } from '@/composables/useRequest'
import { getUser } from '@/api/user'

// 逻辑复用，来源清晰，可传参
const { x, y } = useMouse()
const { data, loading, run } = useRequest(() => getUser(1))
run()
</script>
```

!!! tip
    组合式函数应在 `setup()`（或另一个组合式函数）顶层**同步调用**，不要在 `await` 之后或事件回调里调用，否则生命周期钩子会失效。

## 性能优化 / Performance

### v-once / v-memo

```html
<!-- v-once：只渲染一次，此后整个子树跳过更新（静态内容） -->
<p v-once>© 2026 版权信息</p>

<!-- v-memo：数组内依赖不变则跳过该元素及其子树的更新 -->
<div v-for="item in list" :key="item.id" v-memo="[item.id, item.selected]">
  <p>{{ item.name }}</p>
  <p>{{ item.selected ? '已选' : '未选' }}</p>
</div>
```

### keep-alive 缓存组件

```vue
<template>
  <!-- 缓存路由组件，避免重复创建；include/exclude 按组件 name 过滤 -->
  <router-view v-slot="{ Component }">
    <keep-alive :include="['UserList']" :max="10">
      <component :is="Component" />
    </keep-alive>
  </router-view>
</template>
```

!!! note
    `keep-alive` 内部组件不会触发 `onMounted`/`onUnmounted`，而是走 `onActivated`/`onDeactivated`。

### 懒加载路由

```typescript
// 路由级别：动态 import，按需加载，减小首屏体积
const routes = [
  { path: '/about', component: () => import('@/views/About.vue') },
]
```

```typescript
// 组件级别：异步组件 + 加载态
import { defineAsyncComponent } from 'vue'

const HeavyChart = defineAsyncComponent({
  loader: () => import('@/components/HeavyChart.vue'),
  loadingComponent: Loader,     // 加载中显示
  delay: 200,                   // 延迟显示 loading
  timeout: 5000,                // 超时报错
})
```

### 大列表虚拟滚动

```bash
npm install vue-virtual-scroller
```

```vue
<script setup lang="ts">
import { RecycleScroller } from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

const items = Array.from({ length: 10000 }, (_, i) => ({ id: i, text: `第 ${i} 行` }))
</script>

<template>
  <!-- 只渲染可视区域内的行，万级列表也能流畅滚动 -->
  <RecycleScroller
    :items="items"
    :item-size="50"
    key-field="id"
    class="scroller"
  >
    <template #default="{ item }">
      <div class="row">{{ item.text }}</div>
    </template>
  </RecycleScroller>
</template>

<style scoped>
.scroller { height: 400px; overflow-y: auto; }
.row { height: 50px; line-height: 50px; }
</style>
```

### 优化手段速查

| 手段 | 作用 |
| --- | --- |
| `v-once` | 静态内容只渲染一次 |
| `v-memo` | 依赖不变时跳过子树更新 |
| `keep-alive` | 缓存组件实例，避免反复创建 |
| 路由懒加载 | 按需加载页面，减少首屏体积 |
| 异步组件 | 重组件延迟加载 + loading 态 |
| 虚拟滚动 | 只渲染可视区域，支撑超长列表 |
| `shallowRef` | 大对象整体替换时跳过深层响应式开销 |
| 计算属性缓存 | 用 `computed` 代替模板内复杂表达式 |
| 生产构建 | `npm run build` 生成压缩产物，勿直接部署源码 |

## 常见坑与最佳实践 / Pitfalls & Best Practices

| 问题 | 现象 | 解决 |
| --- | --- | --- |
| 解构丢响应式 | `const { x } = reactive(obj)` 后不更新 | 用 `toRefs` / `storeToRefs` |
| 整体替换 reactive | 赋新对象后失去响应式 | 改回逐个改属性，或整体用 `ref` |
| `v-for` 用 index 作 key | 列表重排时状态错乱 | 用稳定唯一 id 作 `:key` |
| `v-if` 与 `v-for` 同元素 | 取不到循环变量 | 拆成 `<template>` 或先 computed 过滤 |
| 拿不到 DOM | `onMounted` 外访问 ref | DOM 操作放 `onMounted`/`nextTick` |
| watch 不触发 | 对象属性变化未监听 | 加 `deep: true`，或监听 `() => obj.key` |
| 事件未透传 | 子组件根元素拿不到原生事件 | 用 `v-bind="attrs"` 或 `inheritAttrs: false` |
| 子组件方法调不到 | `childRef.value.fn()` 报 undefined | 子组件需 `defineExpose({ fn })` |
| 路由守卫失效 | 刷新后 token 判断异常 | 守卫中避免直接依赖未初始化状态 |
| 首屏慢 | 打包体积过大 | 路由懒加载 + 异步组件 + 按需引入 UI 库 |

!!! tip
    - 组件命名用**多单词 PascalCase**（如 `UserCard`），避免与原生标签冲突。
    - 单向数据流：props 只读，需改动时通过 emit 通知父组件。
    - 优先 `ref` + TS 泛型，类型推导清晰；复杂关联状态再用 `reactive`。
    - 副作用（定时器、监听器）**注册与清理成对出现**在 `onMounted`/`onUnmounted`。
    - 复用逻辑抽成 `use` 开头的组合式函数，替代 Vue 2 的 mixin。
    - 全局状态用 Pinia，跨层级传值用 `provide/inject`，简单父子用 props/emit。

```typescript
// 一个典型页面组件骨架：路由 + 请求 + 副作用清理
import { ref, onMounted, onUnmounted } from 'vue'
import { useRoute } from 'vue-router'
import { useRequest } from '@/composables/useRequest'
import { getUser } from '@/api/user'

const route = useRoute()
const { data: user, loading, run } = useRequest(() => getUser(Number(route.params.id)))

let timer: number
onMounted(() => {
  run()
  timer = window.setInterval(() => console.log('轮询'), 5000)
})
onUnmounted(() => clearInterval(timer))   // 务必清理
```
