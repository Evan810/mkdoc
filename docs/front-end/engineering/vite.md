---
title: Vite
tags:
  - 前端
  - Vite
  - 构建工具
  - 工程化
---

## 介绍 / Introduction

Vite（法语「快」）是基于原生 ES Modules 的新一代前端构建工具，由 Vue 作者尤雨溪主导开发。它把「开发」和「构建」拆成两套引擎：开发时用原生 ESM 按需编译，构建时用 Rollup 打包，从而同时获得极致的冷启动速度与成熟的生产打包能力。

!!! note
    Vite 5 起要求 Node.js 18+ / 20+；Vite 4 要求 Node 14.18+ / 16+。旧项目升级前先确认 Node 版本。

## 核心原理 / Core Principle

| 阶段 | 引擎 | 原理 | 效果 |
| --- | --- | --- | --- |
| 开发（dev） | 原生 ESM + esbuild | 浏览器直接请求模块，服务端按需编译 | 冷启动毫秒级，HMR 精确到模块 |
| 依赖预构建 | esbuild | 把 CJS/UMD 依赖转 ESM 并合并，缓存到 `node_modules/.vite` | 减少请求数，兼容 CommonJS 包 |
| 生产（build） | Rollup | 全量打包、Tree Shaking、代码分割 | 产物成熟稳定，生态兼容好 |

```text
开发：浏览器 --(HTTP)--> Vite Dev Server --(按需转译)--> *.ts / *.vue
构建：源码 --(Rollup 打包)--> dist/ 静态资源
```

!!! tip
    开发环境不打包，因此「冷启动与项目体积几乎无关」；这就是 Vite 比 Webpack 快得多的根本原因。

## 环境要求与创建项目 / Setup & Scaffolding

### 环境要求

| 项目 | 要求 |
| --- | --- |
| Node.js | Vite 5：18+ / 20+；Vite 4：14.18+ / 16+ |
| 包管理器 | npm / pnpm / yarn / bun 均可 |
| 浏览器 | 开发环境需支持原生 ESM（现代浏览器） |

### 脚手架创建

```bash
# 交互式选择框架与语言
npm create vite@latest my-app

# 直接指定模板（避免交互）
npm create vite@latest my-app -- --template react-ts

cd my-app
npm install
npm run dev
```

### 官方模板一览

| 模板名 | 说明 |
| --- | --- |
| `vanilla` / `vanilla-ts` | 原生 JS / TS |
| `vue` / `vue-ts` | Vue 3 |
| `react` / `react-ts` | React |
| `react-swc` / `react-swc-ts` | React + SWC 编译 |
| `preact` / `preact-ts` | Preact |
| `lit` / `lit-ts` | Lit |
| `svelte` / `svelte-ts` | Svelte |
| `solid` / `solid-ts` | Solid |
| `qwik` / `qwik-ts` | Qwik |

!!! note
    `npm create vite@latest` 中的 `--` 用于把参数透传给脚手架，缺少它时 `--template` 可能被 npm 自身吞掉。

## 目录结构与脚本 / Structure & Scripts

```text
my-app/
├── public/                 # 原样拷贝的静态资源（不参与打包）
│   └── favicon.ico
├── src/
│   ├── assets/             # 参与构建的资源（图片、字体等）
│   ├── components/
│   ├── main.tsx            # 入口
│   └── vite-env.d.ts       # Vite 环境类型声明
├── index.html              # 入口 HTML（Vite 的 HTML 即入口）
├── vite.config.ts          # 构建配置
├── tsconfig.json
└── package.json
```

### 常用 npm scripts

```json
{
  "scripts": {
    "dev": "vite",                                    // 启动开发服务器
    "build": "tsc && vite build",                     // 类型检查 + 生产构建
    "build:only": "vite build",                       // 跳过类型检查
    "preview": "vite preview",                        // 本地预览 dist 产物
    "preview:host": "vite preview --host 0.0.0.0"     // 局域网可访问
  }
}
```

| 命令 | 作用 |
| --- | --- |
| `vite` | 启动开发服务器（默认 `http://localhost:5173`） |
| `vite build` | 生产构建，输出到 `dist/` |
| `vite preview` | 启动静态服务器预览 `dist/`，仅用于验证产物 |
| `vite optimize` | 手动触发依赖预构建 |

## 配置文件详解 / Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'node:path';

export default defineConfig({
  plugins: [react()],

  // 部署基础路径：根路径 '/'，子目录 '/my-app/'，CDN 用完整 URL
  base: '/',

  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),        // 路径别名
      '@components': path.resolve(__dirname, 'src/components'),
    },
    // 省略扩展名时按顺序尝试
    extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'],
  },

  server: {
    port: 5173,                                    // 端口，被占用自动 +1
    open: true,                                    // 启动后自动打开浏览器
    host: '0.0.0.0',                               // 局域网可访问（等价 --host）
    strictPort: false,                             // true 时端口占用直接报错
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (p) => p.replace(/^\/api/, ''),   // 去掉 /api 前缀
      },
    },
  },

  build: {
    outDir: 'dist',                                // 输出目录
    assetsDir: 'assets',                           // 静态资源子目录
    sourcemap: false,                              // 生产不建议开启
    minify: 'esbuild',                             // 'esbuild' | 'terser' | false
    target: 'es2015',                              // 浏览器兼容目标（配合 legacy 插件）
    chunkSizeWarningLimit: 1500,                   // 分包体积告警阈值(KB)
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],          // 手动分包
        },
      },
    },
  },

  // 全局常量替换，编译期静态替换
  define: {
    __APP_VERSION__: JSON.stringify('1.0.0'),
  },
});
```

### 关键字段速查

| 字段 | 作用 | 常用取值 |
| --- | --- | --- |
| `base` | 部署根路径 | `'/'`、`'/app/'`、完整 URL |
| `resolve.alias` | 路径别名 | `{ '@': path.resolve(__dirname, 'src') }` |
| `server.port` | 开发端口 | `5173` |
| `server.host` | 监听地址 | `'0.0.0.0'` |
| `server.proxy` | 接口代理 | `{ '/api': { target, changeOrigin } }` |
| `build.outDir` | 产物目录 | `'dist'` |
| `build.assetsDir` | 资源子目录 | `'assets'` |
| `build.rollupOptions` | Rollup 原生配置 | `{ output: { manualChunks } }` |

!!! warning
    `define` 的值会被**直接文本替换**，字符串必须用 `JSON.stringify()` 转义，否则会变成未定义标识符。

## 环境变量与模式 / Env & Modes

```bash
# .env                所有模式共享
# .env.development    仅 dev 模式（npm run dev）
# .env.production     仅 build 模式（npm run build）

VITE_API_BASE=/api
VITE_APP_TITLE=我的学习笔记
```

```typescript
// 使用：只有 VITE_ 前缀的变量才会暴露给客户端
const base = import.meta.env.VITE_API_BASE;
const title = import.meta.env.VITE_APP_TITLE;
const isDev = import.meta.env.DEV;      // 内置：开发模式
const isProd = import.meta.env.PROD;    // 内置：生产模式
const mode = import.meta.env.MODE;      // 当前模式名
```

### 模式与优先级

| 变量 | 说明 |
| --- | --- |
| `import.meta.env.BASE_URL` | 即 `base` 配置值 |
| `import.meta.env.DEV` | 是否开发模式 |
| `import.meta.env.PROD` | 是否生产模式 |
| `import.meta.env.MODE` | 当前 mode（默认 `development`/`production`） |
| `import.meta.env.VITE_*` | 自定义暴露变量 |

!!! warning
    只有以 `VITE_` 开头的变量才会被注入到客户端代码，其他变量仅构建时可用，**切勿把密钥放在 `VITE_` 变量里**（会被打进产物）。

### TypeScript 类型声明

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_BASE: string;
  readonly VITE_APP_TITLE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

!!! tip
    自定义模式可用 `vite build --mode staging` 加载 `.env.staging`；扩展变量时同步更新 `ImportMetaEnv` 才有类型提示。

## 静态资源处理 / Static Assets

| 存放位置 | 处理方式 | 引用方式 |
| --- | --- | --- |
| `public/` | 原样拷贝至产物根目录，不做哈希 | `/favicon.ico`（绝对路径字符串） |
| `src/assets/` | 参与打包，小文件转 base64 | `import logo from './logo.png'` |

```typescript
// 方式一：import 引入（推荐，带哈希、参与构建）
import logo from '@/assets/logo.png';
console.log(logo);                        // 输出打包后的 URL

// 方式二：new URL 动态引用
const img = new URL('./assets/logo.png', import.meta.url).href;

// 方式三：public 目录，用绝对路径
// <img src="/favicon.ico" />
```

### 资源后缀语法

```typescript
// ?raw：以字符串形式导入文件内容
import codeStr from './demo.ts?raw';

// ?url：仅返回资源 URL（不转 base64）
import imgUrl from './logo.png?url';

// ?inline：强制内联 base64
import inlineIcon from './icon.svg?inline';

// ?worker：以 Web Worker 方式导入
import MyWorker from './worker.ts?worker';
```

!!! note
    Vite 默认内联阈值 `assetsInlineLimit` 为 4KB，低于该值的资源会转为 base64 内联，减少请求数。

## CSS 处理 / CSS Handling

### 预处理器（sass / less）

```bash
npm add -D sass          # .scss / .sass
npm add -D less          # .less
npm add -D stylus        # .styl
```

```typescript
// vite.config.ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        // 全局注入变量文件，免去每个文件重复 @use
        additionalData: `@use "@/styles/variables.scss" as *;`,
      },
    },
  },
});
```

### CSS Modules

```typescript
// 文件名以 .module.css / .module.scss 结尾即自动启用 CSS Modules
import styles from './button.module.css';

// <button className={styles.primary} />
```

```typescript
// 自定义 Modules 命名规则
export default defineConfig({
  css: {
    modules: {
      localsConvention: 'camelCaseOnly',   // kebab-case 转 camelCase
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
  },
});
```

### PostCSS

```javascript
// postcss.config.js（Vite 自动识别）
export default {
  plugins: {
    'postcss-px-to-viewport': { viewportWidth: 375 },
    autoprefixer: {},
  },
};
```

!!! tip
    全局样式变量统一放 `src/styles/`，通过 `additionalData` 注入；注意该方式会重复注入到每个文件，增加构建量。

## 常用插件 / Plugins

| 插件 | 作用 |
| --- | --- |
| `@vitejs/plugin-react` | React 支持（Babel 编译 + Fast Refresh） |
| `@vitejs/plugin-react-swc` | React + SWC，编译更快 |
| `@vitejs/plugin-vue` | Vue 3 单文件组件 |
| `@vitejs/plugin-legacy` | 为旧浏览器生成 legacy 兼容产物 |
| `vite-plugin-compression` | 产出 gzip / brotli 压缩文件 |
| `unplugin-auto-import` | 自动导入 API，免手写 import |
| `unplugin-vue-components` | 按需自动注册组件 |
| `vite-plugin-svg-icons` | SVG 雪碧图 |
| `rollup-plugin-visualizer` | 产物体积可视化分析 |

```typescript
import react from '@vitejs/plugin-react';
import legacy from '@vitejs/plugin-legacy';
import compression from 'vite-plugin-compression';
import AutoImport from 'unplugin-auto-import/vite';
import Components from 'unplugin-vue-components/vite';
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers';

export default defineConfig({
  plugins: [
    react(),
    // 旧浏览器兼容：生成 legacy 包 + polyfill
    legacy({
      targets: ['defaults', 'not IE 11'],
    }),
    // 产出 .gz 文件，交由 Nginx gzip_static 直接返回
    compression({ algorithm: 'gzip', threshold: 10240 }),
    compression({ algorithm: 'brotliCompress', ext: '.br' }),
    // 自动导入：免手写 ref、computed 等
    AutoImport({ imports: ['vue', 'react'], resolvers: [ElementPlusResolver()] }),
    // 自动按需注册组件
    Components({ resolvers: [ElementPlusResolver()] }),
  ],
});
```

!!! note
    `@vitejs/plugin-react-swc` 使用 SWC 替代 Babel，冷启动与 HMR 更快，但部分 Babel 插件生态不兼容。

## 开发代理 / Dev Proxy

```typescript
export default defineConfig({
  server: {
    proxy: {
      // 场景一：前缀保留，直接转发
      '/api': 'http://localhost:8080',

      // 场景二：去掉前缀（后端无 /api）
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },

      // 场景三：WebSocket 代理
      '/ws': {
        target: 'ws://localhost:9000',
        ws: true,
        changeOrigin: true,
      },
    },
  },
});
```

| 选项 | 说明 |
| --- | --- |
| `target` | 后端服务地址 |
| `changeOrigin` | 修改请求头的 Host，避免跨域校验失败 |
| `rewrite` | 重写请求路径 |
| `ws` | 是否代理 WebSocket |
| `secure` | https 目标是否校验证书 |

!!! warning
    代理只在**开发环境**生效；生产环境需由 Nginx 等网关配置反向代理，二者不可混淆。

## 构建优化 / Build Optimization

### 手动分包 manualChunks

```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['antd', '@ant-design/icons'],
          'utils-vendor': ['lodash-es', 'dayjs'],
        },
      },
    },
  },
});
```

```typescript
// 函数式分包：按依赖路径自动归类
manualChunks(id) {
  if (id.includes('node_modules')) {
    return id.split('node_modules/')[1].split('/')[0].toString();
  }
}
```

### gzip / brotli 压缩

```bash
# 用 compression 插件在构建时产出压缩文件（推荐配合 Nginx gzip_static on）
npm add -D vite-plugin-compression
```

```nginx
# Nginx：优先返回预压缩文件
gzip_static on;
brotli_static on;
```

### 产物分析

```typescript
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'dist/stats.html',   // 生成的报告路径
      gzipSize: true,                // 显示 gzip 体积
      brotliSize: true,
      open: true,                    // 构建后自动打开
    }),
  ],
});
```

| 优化手段 | 收益 |
| --- | --- |
| `manualChunks` 分包 | 利用浏览器缓存，减少重复加载 |
| `compression` 压缩 | 传输体积下降 60%+ |
| `sourcemap: false` | 减小产物体积、加快构建 |
| `build.target` 提升 | 减少降级转译，产物更小 |
| 按需引入组件库 | 显著减小首屏 JS |

!!! tip
    优先做「按需引入 + 分包」，再做压缩；`brotli` 体积最小但需服务端支持。

## HMR 原理与调试 / HMR

原理：文件变更 → Vite 监听 → 定位模块图中的边界模块 → 通过 WebSocket 推送更新 → 浏览器仅替换该模块并执行 `accept` 回调，**不刷新整页**。

```typescript
// 自定义 HMR 处理（组件内部）
if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    // 模块热更新时执行
    newModule.render();
  });

  import.meta.hot.dispose(() => {
    // 清理副作用：解绑事件、清除定时器
    clearInterval(timer);
  });
}
```

| 现象 | 说明 |
| --- | --- |
| 页面整页刷新 | 模块未处理 `accept`，HMR 边界提升失败 |
| 修改样式即时生效 | CSS 走独立 HMR 通道 |
| 修改 `vite.config.ts` | 需手动重启 dev server |

!!! note
    修改环境变量 `.env` 文件后必须重启 dev server，Vite 不会热更新其内容。

## Vite 与 Webpack 对比 / Vite vs Webpack

| 对比项 | Vite | Webpack |
| --- | --- | --- |
| 开发启动 | 基于原生 ESM，按需编译，毫秒级 | 需全量打包，秒级~分钟级 |
| HMR 速度 | 与模块数量弱相关，极快 | 随项目增大而变慢 |
| 生产构建 | Rollup | 自研打包器 |
| 配置复杂度 | 低，约定优于配置 | 高，loader/plugin 组合灵活 |
| 生态 | 快速增长，现代框架首选 | 极其成熟，历史插件海量 |
| 兼容性 | 现代浏览器，旧浏览器需 legacy 插件 | 兼容性极强 |
| 适用场景 | 新项目、现代浏览器、快速迭代 | 复杂/存量/legacy 项目、深度定制 |

```bash
# 一句话选型
# 新项目 → Vite；存量复杂项目或需深度定制 → Webpack
```

!!! tip
    二者并非对立：存量项目可先用 Webpack 保持稳定，新模块或微前端子应用迁移到 Vite。

## 常见问题 / FAQ

| 问题 | 原因 | 解决 |
| --- | --- | --- |
| 端口被占用 | 5173 已被使用 | 改 `server.port` 或设 `strictPort: true` |
| 白屏 / 模块 404 | `base` 与实际部署路径不一致 | 设置正确的 `base`（如 `'/app/'`） |
| 刷新 404 | 前端路由未配置 fallback | 部署时 Nginx `try_files $uri /index.html` |
| 环境变量不生效 | 未加 `VITE_` 前缀 | 变量名以 `VITE_` 开头 |
| 别名不生效 | 只配了 Vite 未配 TS | 同步配置 `tsconfig.json` 的 `paths` |
| CommonJS 报错 | 依赖未被预构建 | 加入 `optimizeDeps.include` |

```json
// tsconfig.json：让编辑器与 Vite 别名保持一致
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

```typescript
// 处理未被预构建的依赖报错
export default defineConfig({
  optimizeDeps: {
    include: ['some-cjs-package'],
  },
});
```

```nginx
# SPA 部署：Nginx 兜底到 index.html
location / {
  try_files $uri $uri/ /index.html;
}
```

!!! warning
    `vite preview` 仅用于本地验证产物，**不要用于生产**；生产请用 Nginx / CDN 托管 `dist/`。

!!! tip
    - 别名务必 `vite.config.ts` 与 `tsconfig.json` 两处同步配置。
    - 静态资源优先 `import`，需要固定路径的（如 favicon）放 `public/`。
    - 首屏体积告警时先查组件库是否按需引入，再考虑分包与压缩。
