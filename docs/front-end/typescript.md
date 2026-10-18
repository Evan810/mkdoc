---
title: TypeScript
tags:
  - 前端
  - TypeScript
  - 类型系统
  - 泛型
---

## 介绍 / Introduction

TypeScript 是 JavaScript 的**超集**，在 JS 之上引入静态类型系统：类型错误在**编译期**就被发现，而不是等到运行时。编译产物仍是标准 JS，可跑在任何支持 JS 的环境。

!!! note
    类型只在开发/编译期存在，编译后会**擦除**（type erasure），不影响运行时性能。推荐配合 ESLint + Prettier 使用。

## 配置与编译 / Config & Compile

```bash
# 安装
npm install -D typescript
npm install -g typescript

# 常用命令
tsc --init          # 生成 tsconfig.json
tsc                 # 按 tsconfig 编译整个项目
tsc app.ts          # 编译单文件
npx tsc --noEmit    # 只做类型检查，不产出文件（CI 常用）
tsc --watch         # 监听模式
```

### tsconfig.json 常用配置速查

```json
{
  "compilerOptions": {
    "target": "ES2020",              // 编译目标 JS 版本
    "module": "ESNext",              // 模块系统
    "moduleResolution": "Bundler",   // 模块解析策略
    "lib": ["ES2020", "DOM"],        // 引入哪些内置类型库
    "jsx": "react-jsx",              // JSX 处理方式
    "strict": true,                  // 开启全部严格检查（强烈建议）
    "noImplicitAny": true,           // 禁止隐式 any
    "strictNullChecks": true,        // null/undefined 严格区分
    "esModuleInterop": true,         // 兼容 CommonJS 默认导入
    "skipLibCheck": true,            // 跳过 .d.ts 类型检查，加速
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,       // 允许 import json
    "isolatedModules": true,
    "outDir": "./dist",              // 输出目录
    "rootDir": "./src",              // 源码根目录
    "declaration": true,             // 生成 .d.ts
    "sourceMap": true,
    "baseUrl": ".",
    "paths": {                       // 路径别名
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

| 配置项 | 作用 | 建议 |
| --- | --- | --- |
| `strict` | 一键开启全部严格模式 | 新项目必开 |
| `strictNullChecks` | `null`/`undefined` 不可赋给其他类型 | 必开 |
| `noImplicitAny` | 隐式 `any` 报错 | 必开 |
| `target` | 输出语法版本 | 现代浏览器用 `ES2020+` |
| `esModuleInterop` | 简化默认导入 | 一般开 |
| `skipLibCheck` | 跳过依赖类型检查 | 通常开，加速构建 |

!!! warning
    不要用 `any` 图省事躲过报错，那等于关掉了类型保护。真需要宽泛类型时用 `unknown` 再收窄。

## 基础类型 / Basic Types

```typescript
// 原始类型
let str: string = 'hello';
let num: number = 42;
let bool: boolean = true;
let big: bigint = 100n;
let sym: symbol = Symbol('k');

// 数组
let nums: number[] = [1, 2, 3];
let strs: Array<string> = ['a', 'b'];

// 元组：定长、定类型
let tuple: [string, number] = ['age', 18];

// 对象
let obj: { name: string; age?: number } = { name: 'Tom' };

// 函数
let fn: (a: number, b: number) => number = (a, b) => a + b;

// 联合 / 字面量
let id: string | number = 1;
type Dir = 'up' | 'down' | 'left' | 'right';
```

### 类型注解 vs 类型推断

```typescript
// 显式注解：变量声明处写类型
let name: string = 'Tom';

// 类型推断：TS 自动推断，无需手写
let age = 18;                 // 推断为 number
const list = [1, 2, 3];       // 推断为 number[]
```

!!! tip
    局部变量能推断就交给推断，只在**函数参数、返回值、复杂结构、公共 API**处显式标注，代码更简洁。

### 特殊类型：any / unknown / never / void / object

| 类型 | 含义 | 说明 |
| --- | --- | --- |
| `any` | 任意类型 | 放弃类型检查，尽量少用 |
| `unknown` | 未知类型 | 安全的 any，用前必须收窄 |
| `never` | 永不存在的值 | 抛错函数的返回、穷尽检查兜底 |
| `void` | 无返回值 | 函数无 return 时 |
| `object` | 非原始类型 | 任意对象，不能访问具体属性 |
| `null` / `undefined` | 空值 | `strict` 下需显式处理 |

```typescript
// unknown：用前必须收窄
function parse(input: unknown) {
  // input.toUpperCase();               // 报错：unknown 不可直接调用
  if (typeof input === 'string') {
    return input.toUpperCase();          // 收窄后可用
  }
}

// never
function fail(msg: string): never {
  throw new Error(msg);
}

// void
function log(msg: string): void {
  console.log(msg);
}
```

## 接口与类型别名 / Interface & Type

```typescript
// interface：定义对象结构
interface User {
  id: number;
  name: string;
  email?: string;         // 可选属性
  readonly createdAt: Date; // 只读
}

// type：可表示联合、元组、原始类型等更广
type ID = string | number;
type Point = { x: number; y: number };
type Callback = (data: string) => void;
```

| 维度 | interface | type |
| --- | --- | --- |
| 描述对象 | 可以 | 可以 |
| 声明合并 | 支持（同名自动合并） | 不支持 |
| 联合/交叉/元组 | 需间接 | 直接写 |
| 实现 implements | 可以 | 可以（对象类型）|
| 扩展 | `extends` | `&` 交叉 |
| 推荐 | 定义对象/公开 API | 联合、工具、组合 |

```typescript
// interface 扩展
interface Admin extends User {
  role: string;
}

// type 组合
type Admin = User & { role: string };

// 声明合并（interface 特有）
interface Window { myGlobal: string; }
interface Window { myOther: number; }
```

## 联合、交叉与字面量 / Union & Intersection

```typescript
// 联合：满足其一即可
type Result = { ok: true; data: string } | { ok: false; error: string };

// 交叉：同时满足
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

// 字面量类型：限定取值
type Status = 'pending' | 'success' | 'error';
let s: Status = 'success';
// s = 'done';   // 报错

// 可选与只读
interface Config {
  readonly host: string;      // 只读
  port?: number;              // 可选
}
```

!!! tip
    联合类型收窄后，TS 会通过**可辨识联合（discriminated union）**自动排除不可能分支，结合 `switch` 做穷尽检查非常安全。

```typescript
function handle(res: Result) {
  if (res.ok) {
    console.log(res.data);     // 此分支 TS 知道是成功态
  } else {
    console.error(res.error);  // 此分支知道是失败态
  }
}
```

## 函数 / Functions

```typescript
// 基本签名
function add(a: number, b: number): number {
  return a + b;
}

// 可选参数（必须在必选参数之后）
function greet(name: string, title?: string) {
  return title ? `${title} ${name}` : name;
}

// 默认参数
function pow(base: number, exp = 2) {
  return base ** exp;
}

// 剩余参数
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}

// 函数类型（箭头）
const mul: (a: number, b: number) => number = (a, b) => a * b;

// 函数重载：多个签名 + 一个实现
function toArray(value: string): string[];
function toArray(value: number): number[];
function toArray(value: string | number): (string | number)[] {
  return Array.isArray(value) ? value : [value];
}
```

!!! note
    箭头函数与普通函数类型的区别：需要 `this` 类型时用 `(this: T, ...) => void`；方法内可用 `this` 参数显式声明类型。

## 泛型 / Generics

### 泛型函数

```typescript
// 保持入参和返回值类型一致
function identity<T>(value: T): T {
  return value;
}

const a = identity<string>('hi');   // 也可让 TS 自动推断
const b = identity(42);             // 推断为 number

// 多个类型参数
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}
```

### 泛型接口 / 类型别名

```typescript
interface ApiResponse<T> {
  code: number;
  data: T;
  message: string;
}

const res: ApiResponse<User[]> = { code: 0, data: [], message: 'ok' };

type Box<T> = { value: T };
```

### 泛型类

```typescript
class Stack<T> {
  private items: T[] = [];
  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}

const s = new Stack<number>();
s.push(1);
```

### 泛型约束（extends）

```typescript
// 约束 T 必须有 length 属性
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}

// keyof 约束：K 必须是 T 的键
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: 'Tom' };
const name = get(user, 'name');   // 类型为 string
```

### 默认类型参数

```typescript
interface Pager<T = unknown> {
  list: T[];
  total: number;
}

const p1: Pager = { list: [], total: 0 };        // 用默认
const p2: Pager<User> = { list: [], total: 0 };  // 指定
```

!!! tip
    泛型是「类型的参数」。当你发现两个地方的类型应该互相绑定，或你想写一个对多种类型都工作的函数/容器，就该用泛型。

## 枚举 / Enums

```typescript
// 数字枚举：自动递增
enum Status {
  Pending,   // 0
  Success,   // 1
  Failed,    // 2
}

// 字符串枚举：可读性好、调试友好
enum Role {
  Admin = 'admin',
  User = 'user',
}

// 常量枚举：编译期内联，减少产物
const enum Color {
  Red = 'red',
  Blue = 'blue',
}
```

| 方案 | 产物 | 优点 | 缺点 |
| --- | --- | --- | --- |
| `enum` | 生成运行时对象 | 类型 + 值一体 | 增加产物，跨模块麻烦 |
| `const enum` | 编译期内联 | 无运行时开销 | 受 `isolatedModules` 限制 |
| 字面量联合 + `as const` | 纯类型 | 零产物、灵活 | 需手动维护 |

```typescript
// 现代推荐：用 as const 对象 + 联合类型替代 enum
const Status = {
  Pending: 'pending',
  Success: 'success',
  Failed: 'failed',
} as const;

type Status = (typeof Status)[keyof typeof Status];  // 'pending' | 'success' | 'failed'
```

## 类型断言与守卫 / Assertions & Guards

```typescript
// 1. as 断言：告诉 TS「我知道它是什么」
const el = document.getElementById('app') as HTMLDivElement;
const len = (value as string).length;

// 2. 非空断言 !：排除 null/undefined
const node = document.querySelector('.box')!;
node.innerHTML = 'hi';

// 3. typeof 守卫（原始类型）
function fmt(x: string | number) {
  if (typeof x === 'string') return x.toUpperCase();
  return x.toFixed(2);
}

// 4. instanceof 守卫（类实例）
function fmtDate(d: Date | string) {
  if (d instanceof Date) return d.getTime();
  return new Date(d).getTime();
}

// 5. in 守卫（属性存在性）
type Fish = { swim: () => void };
type Bird = { fly: () => void };
function move(animal: Fish | Bird) {
  if ('swim' in animal) animal.swim();
  else animal.fly();
}

// 6. 自定义类型守卫：返回 x is T
function isString(x: unknown): x is string {
  return typeof x === 'string';
}
```

!!! warning
    `as` 和 `!` 是「你说服编译器」，编译器**不会帮你验证**。断言错方向会在运行时爆炸，优先用类型守卫做真实收窄。

## 工具类型速查 / Utility Types

| 工具类型 | 作用 | 示例结果 |
| --- | --- | --- |
| `Partial<T>` | 全部属性变可选 | `{ a?: 1 }` |
| `Required<T>` | 全部属性变必选 | `{ a: 1 }` |
| `Readonly<T>` | 全部属性变只读 | 不可重新赋值 |
| `Pick<T, K>` | 挑选部分属性 | `Pick<User,'id'>` |
| `Omit<T, K>` | 剔除部分属性 | `Omit<User,'pwd'>` |
| `Record<K, V>` | 构造键值映射 | `Record<string, number>` |
| `Exclude<T, U>` | 从联合中排除 | `Exclude<'a'\|'b','a'>` → `'b'` |
| `Extract<T, U>` | 从联合中提取 | `Extract<'a'\|'b','a'>` → `'a'` |
| `ReturnType<F>` | 函数返回值类型 | `ReturnType<typeof fn>` |
| `Parameters<F>` | 函数参数元组类型 | `[string, number]` |
| `Awaited<T>` | 递归解包 Promise | `Awaited<Promise<string>>` → `string` |
| `NonNullable<T>` | 去除 null/undefined | |

```typescript
interface User {
  id: number;
  name: string;
  password: string;
}

type PartialUser = Partial<User>;              // 全可选
type UserPreview = Pick<User, 'id' | 'name'>;  // 只保留 id/name
type SafeUser = Omit<User, 'password'>;        // 去掉密码
type UserMap = Record<string, User>;           // 字典

// 函数相关
async function getUser() {
  return { id: 1, name: 'Tom' };
}
type UserResult = Awaited<ReturnType<typeof getUser>>;  // { id: number; name: string }

// Exclude / Extract
type T0 = Exclude<'a' | 'b' | 'c', 'a'>;       // 'b' | 'c'
type T1 = Extract<'a' | 'b' | 'c', 'a' | 'c'>; // 'a' | 'c'
```

!!! tip
    工具类型的实现本质是**映射类型 + 条件类型**。理解 `Partial` 的写法 `{ [K in keyof T]?: T[K] }`，你就能自己写自定义工具类型。

```typescript
// 自定义工具类型：把所有属性变为可空
type Nullable<T> = { [K in keyof T]: T[K] | null };

// 自定义：把指定键变为可选
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
```

## 类与访问修饰符 / Classes

```typescript
class Animal {
  public name: string;            // 公开，默认
  private secret = 'shh';         // 仅本类可访问
  protected age: number;          // 本类及子类可访问
  readonly kind = 'animal';       // 只读，初始化后不可改

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  // 参数属性：构造函数参数前加修饰符，自动声明并赋值
  // constructor(public name: string, private age: number) {}

  speak(): void {
    console.log(`${this.name} speaks`);
  }

  // 静态成员
  static create(name: string): Animal {
    return new Animal(name, 0);
  }
}
```

```typescript
// 继承
class Dog extends Animal {
  constructor(name: string) {
    super(name, 3);             // 必须先调用 super
  }
  speak(): void {
    console.log('Woof');        // 重写父类方法
  }
}

// 抽象类：不能直接实例化，供子类实现
abstract class Shape {
  abstract area(): number;      // 子类必须实现
  describe(): string {
    return `面积 ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private r: number) { super(); }
  area(): number { return Math.PI * this.r ** 2; }
}

// 接口实现
interface Serializable {
  serialize(): string;
}
class Doc implements Serializable {
  serialize(): string { return JSON.stringify(this); }
}
```

| 修饰符 | 本类 | 子类 | 外部 |
| --- | --- | --- | --- |
| `public`（默认） | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |
| `readonly` | 只读 | 只读 | 只读 |

!!! note
    TypeScript 的 `private` 是**编译期**约束（`#field` 才是运行时私有）。需要真正的运行时限定时，用 ES 私有字段 `#name`。

## 模块、声明与命名空间 / Modules & Declarations

```typescript
// 命名导出
export const PI = 3.14;
export function area(r: number) { return PI * r ** 2; }

// 默认导出
export default class Calculator {}

// 导入
import Calculator, { PI, area } from './math';
import * as MathUtils from './math';
import type { User } from './types';    // 仅类型导入，编译后删除
```

```typescript
// .d.ts：类型声明文件，只写类型不给实现
// types/user.d.ts
declare interface User {
  id: number;
  name: string;
}

// 声明全局变量/模块
declare const VERSION: string;
declare module '*.svg' {
  const content: string;
  export default content;
}
```

```typescript
// 命名空间（老式组织方式，新代码优先用模块）
namespace Validation {
  export function isEmail(s: string): boolean {
    return /^[^@]+@[^@]+$/.test(s);
  }
}
Validation.isEmail('a@b.com');
```

!!! tip
    第三方库没有类型时，安装 `@types/xxx`（如 `@types/node`）；仍缺类型时，自己写 `.d.ts` 用 `declare module 'lib'` 兜底。

## 与 React / Vue 结合 / With Frameworks

```tsx
// React 常见类型
import type { FC, ReactNode, PropsWithChildren, ChangeEvent } from 'react';
import { useState } from 'react';

interface ButtonProps {
  label: string;
  children?: ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'ghost';
}

// 函数组件：推荐直接标注 props 类型
function Button({ label, children, onClick, variant = 'primary' }: ButtonProps) {
  return <button onClick={onClick}>{label}{children}</button>;
}

// 泛型组件
interface ListProps<T> {
  items: T[];
  render: (item: T) => ReactNode;
}
function List<T>({ items, render }: ListProps<T>) {
  return <ul>{items.map((item, i) => <li key={i}>{render(item)}</li>)}</ul>;
}

// 事件类型
const handleChange = (e: ChangeEvent<HTMLInputElement>) => setValue(e.target.value);

// 常用 ref / 状态类型
const [user, setUser] = useState<User | null>(null);
```

```typescript
// Vue 3 常见类型
import type { Ref, ComputedRef } from 'vue';
import { ref, computed, defineProps } from 'vue';

interface Props {
  title: string;
  count?: number;
}
const props = defineProps<Props>();

const total = ref<number>(0);                         // Ref<number>
const double = computed<number>(() => total.value * 2); // ComputedRef<number>

// 显式泛型 ref
const list = ref<User[]>([]);
```

## 常见模式与最佳实践 / Best Practices

| 实践 | 说明 |
| --- | --- |
| 开启 `strict` | 换来最强的类型安全 |
| 优先 `unknown` 而非 `any` | 强制收窄，避免失控 |
| 类型守卫替代断言 | `as`/`!` 只在确知时用 |
| 对象用 `interface`，联合用 `type` | 各取所长 |
| 用 `as const` 替代 `enum` | 零运行时开销、更灵活 |
| 泛型约束用 `extends` | 让泛型可控且有提示 |
| 用工具类型组合 | 减少重复类型定义 |
| 公共 API 显式标注 | 私有局部交给推断 |
| 用 `import type` | 让仅类型导入在编译后删除 |

```typescript
// 例：可辨识联合建模 UI 状态
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User[] }
  | { status: 'error'; message: string };

function render(state: State) {
  switch (state.status) {
    case 'loading':
      return '加载中...';
    case 'success':
      return state.data.length + ' 条';   // TS 知道有 data
    case 'error':
      return state.message;                // TS 知道有 message
    default:
      return '';
  }
}
```

!!! tip
    - 不要给每个变量都写类型：能推断就交给推断，只标注「边界」（函数签名、公共 API）。
    - 类型报错别急着 `@ts-ignore`，先想想是类型写错了还是逻辑写错了；确需绕过用 `@ts-expect-error` 并附原因。
    - 大型项目把类型集中在 `types/` 或 `*.d.ts`，团队成员共用一套契约。
