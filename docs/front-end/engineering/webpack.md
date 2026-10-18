---
title: Webpack
tags:
  - 前端
  - Webpack
  - 构建工具
  - 工程化
---

## 介绍 / Introduction

Webpack 是老牌模块打包器（module bundler），把项目中的 JS、CSS、图片、字体等一切资源都视为「模块」，从入口出发递归构建依赖图，再统一打包为浏览器可运行的产物。它拥有极为成熟的 loader/plugin 生态，是复杂工程与存量项目的首选。

!!! note
    新项目推荐优先使用 Vite；Webpack 在 legacy 存量项目中仍被广泛使用，也常用于需要高度定制打包流程的场景。

## 核心概念 / Core Concepts

| 概念 | 作用 | 示例 |
| --- | --- | --- |
| `entry` | 打包入口，依赖图的起点 | `./src/index.js` |
| `output` | 产物输出位置与命名 | `path` + `filename` |
| `loader` | 把非 JS 资源转成可打包的模块 | `babel-loader`、`css-loader` |
| `plugin` | 干预构建全流程，做增强处理 | `HtmlWebpackPlugin` |
| `mode` | 内置优化开关 | `development` / `production` |
| `devtool` | source map 生成策略 | `source-map`、`eval-cheap-module-source-map` |
| `resolve` | 模块解析规则 | `alias`、`extensions` |
| `optimization` | 分包、压缩、Tree Shaking | `splitChunks` |

```text
entry --> [loader 转换] --> 模块依赖图 --> [plugin 增强] --> output 产物
```

!!! tip
    loader 负责「翻译单个文件」，plugin 负责「在整个构建流程中做事情」，二者职责不同，别混用。

## 配置文件与多环境 / Config & Multi-Env

### 最小可用配置

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    clean: true,                       // 构建前清空输出目录
  },
  module: {
    rules: [
      { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ },
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },
    ],
  },
};
```

### 多环境拆分（推荐）

```text
config/
├── webpack.common.js     # 公共配置
├── webpack.dev.js        # 开发配置
└── webpack.prod.js       # 生产配置
```

```javascript
// webpack.common.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    clean: true,
  },
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    alias: { '@': path.resolve(__dirname, '../src') },
  },
  module: {
    rules: [
      { test: /\.(js|jsx|ts|tsx)$/, use: 'babel-loader', exclude: /node_modules/ },
      {
        test: /\.(png|jpe?g|gif|svg|woff2?|eot|ttf)$/,
        type: 'asset',
        parser: { dataUrlCondition: { maxSize: 4 * 1024 } },   // <4KB 内联
        generator: { filename: 'assets/[hash][ext][query]' },
      },
    ],
  },
  plugins: [new HtmlWebpackPlugin({ template: './public/index.html' })],
};
```

```javascript
// webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval-cheap-module-source-map',     // 开发用快而轻的 source map
  devServer: {
    port: 3000,
    hot: true,                                 // 开启 HMR
    open: true,
    historyApiFallback: true,                  // SPA 路由回退
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
      },
    },
  },
});
```

```javascript
// webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
  output: {
    filename: 'js/[name].[contenthash:8].js',
    chunkFilename: 'js/[name].[contenthash:8].chunk.js',
  },
  optimization: {
    minimize: true,
    minimizer: ['...', new CssMinimizerPlugin()],  // '...' 保留默认 Terser
    splitChunks: { chunks: 'all' },
    runtimeChunk: 'single',
  },
  plugins: [
    new MiniCssExtractPlugin({ filename: 'css/[name].[contenthash:8].css' }),
  ],
});
```

```json
// package.json scripts
{
  "scripts": {
    "dev": "webpack serve --config config/webpack.dev.js",
    "build": "webpack --config config/webpack.prod.js"
  }
}
```

!!! tip
    多环境拆分的关键是 `webpack-merge` 的 `merge` 函数；数组类配置（如 `plugins`、`rules`）会被合并而非覆盖。

| 环境 | devtool | 优化 | 输出命名 |
| --- | --- | --- | --- |
| 开发 | `eval-cheap-module-source-map` | 不压缩 | 固定名 |
| 生产 | `source-map` / `hidden-source-map` | 压缩 + 分包 | `[contenthash]` |

## 常用 loader / Loaders

| loader | 作用 |
| --- | --- |
| `babel-loader` | 转译 ES 新语法 / JSX，配合 `@babel/preset-env` |
| `ts-loader` / `babel-loader` + `@babel/preset-typescript` | 编译 TypeScript |
| `css-loader` | 解析 CSS 中的 `@import`、`url()` |
| `style-loader` | 把 CSS 注入 `<style>`（开发用） |
| `MiniCssExtractPlugin.loader` | 抽取独立 CSS 文件（生产用） |
| `sass-loader` / `less-loader` | 编译预处理器 |
| `postcss-loader` | 后处理（autoprefixer 等） |
| `esbuild-loader` | 用 esbuild 加速 JS/TS 转译 |
| `thread-loader` | 多线程执行 loader，提升构建速度 |

### JS / TS 转译

```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', { targets: 'defaults', useBuiltIns: 'usage', corejs: 3 }],
    ['@babel/preset-react', { runtime: 'automatic' }],
    '@babel/preset-typescript',
  ],
  plugins: ['@babel/plugin-transform-runtime'],
};
```

```javascript
// webpack rule
{
  test: /\.(js|jsx|ts|tsx)$/,
  exclude: /node_modules/,
  use: {
    loader: 'babel-loader',
    options: { cacheDirectory: true },   // 开启缓存，加速二次构建
  },
}
```

!!! note
    `ts-loader` 默认做类型检查（较慢），只做转译可加 `transpileOnly: true`，类型检查交给 `fork-ts-checker-webpack-plugin` 并行执行。

### CSS 链路（注意顺序：从右到左）

```javascript
{
  test: /\.scss$/,
  use: [
    isProd ? MiniCssExtractPlugin.loader : 'style-loader',  // 3. 注入
    { loader: 'css-loader', options: { modules: { localIdentName: '[name]__[local]--[hash:base64:5]' } } }, // 2. 解析
    'postcss-loader',                                        // 1.5 后处理
    'sass-loader',                                           // 1. 编译
  ],
}
```

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    'postcss-px-to-viewport': { viewportWidth: 375 },
    autoprefixer: {},
  },
};
```

!!! warning
    loader 数组**从后往前**执行：`sass-loader → postcss-loader → css-loader → style-loader`，顺序写反会报错。

### 资源模块（asset modules）

```javascript
module.exports = {
  module: {
    rules: [
      // asset/resource：输出独立文件（替代 file-loader）
      { test: /\.png$/, type: 'asset/resource' },

      // asset/inline：转 base64 内联（替代 url-loader）
      { test: /\.svg$/, type: 'asset/inline' },

      // asset/source：以字符串读取源内容（替代 raw-loader）
      { test: /\.txt$/, type: 'asset/source' },

      // asset：自动按大小决定内联或输出（替代 url-loader 阈值）
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: 'asset',
        parser: { dataUrlCondition: { maxSize: 8 * 1024 } },
        generator: { filename: 'assets/[hash][ext][query]' },
      },
    ],
  },
};
```

| 旧 loader | 新写法（Webpack 5） |
| --- | --- |
| `file-loader` | `type: 'asset/resource'` |
| `url-loader` | `type: 'asset'` + `dataUrlCondition.maxSize` |
| `raw-loader` | `type: 'asset/source'` |

!!! tip
    Webpack 5 内置资源模块，**不再需要安装 `file-loader` / `url-loader` / `raw-loader`**，直接使用 `type` 即可。

## 常用 plugin / Plugins

| plugin | 作用 |
| --- | --- |
| `HtmlWebpackPlugin` | 以模板生成 HTML，自动注入打包后的 JS/CSS |
| `MiniCssExtractPlugin` | 抽取 CSS 为独立文件 |
| `CopyWebpackPlugin` | 拷贝静态资源到产物目录 |
| `DefinePlugin` | 编译期替换全局常量 |
| `CleanWebpackPlugin` | 清理输出目录（Webpack 5 可用 `output.clean`） |
| `BundleAnalyzerPlugin` | 产物体积可视化分析 |
| `ForkTsCheckerWebpackPlugin` | 独立进程做 TS 类型检查 |
| `CompressionWebpackPlugin` | 生成 gzip / brotli 文件 |

```javascript
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  plugins: [
    // 生成 HTML 并自动注入资源
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: { collapseWhitespace: true, removeComments: true },
    }),
    // 抽取 CSS
    new MiniCssExtractPlugin({ filename: 'css/[name].[contenthash:8].css' }),
    // 拷贝 public 下的静态文件
    new CopyWebpackPlugin({
      patterns: [{ from: path.resolve(__dirname, 'public'), to: path.resolve(__dirname, 'dist') }],
    }),
    // 注入全局常量
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      __APP_VERSION__: JSON.stringify('1.0.0'),
    }),
    // 产物分析（按需开启）
    // new BundleAnalyzerPlugin({ analyzerMode: 'static', openAnalyzer: true }),
  ],
};
```

!!! warning
    `DefinePlugin` 的值必须是**已序列化的代码字符串**，如 `JSON.stringify('1.0.0')`；直接写 `'1.0.0'` 会被当成表达式。

## devServer 配置 / Dev Server

```javascript
module.exports = {
  devServer: {
    port: 3000,                          // 端口
    hot: true,                           // 开启 HMR
    open: true,                          // 自动打开浏览器
    host: '0.0.0.0',                     // 局域网可访问
    compress: true,                      // 启用 gzip
    static: { directory: path.join(__dirname, 'public') },  // 静态目录
    historyApiFallback: true,            // SPA 路由回退到 index.html
    client: { overlay: { errors: true, warnings: false } }, // 报错覆盖层
    proxy: [
      {
        context: ['/api'],
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
      },
    ],
  },
};
```

| 选项 | 说明 |
| --- | --- |
| `port` / `host` | 监听端口与地址 |
| `hot` | 启用模块热替换 |
| `open` | 启动后打开浏览器 |
| `historyApiFallback` | 前端路由刷新 404 兜底 |
| `proxy` | 接口代理，解决开发跨域 |
| `static` | 静态资源目录 |

!!! note
    历史配置 `devServer` 放在 `webpack.config.js` 顶层；Webpack 5 中 `proxy` 推荐用数组写法（`context` + `target`）。

## 模块热替换 / HMR

原理：文件变更 → webpack 重新编译受影响的 chunk → 通过 `webpack-dev-server` 的 WebSocket 通知浏览器 → 浏览器拉取更新模块 → 运行时用新模块替换旧模块，**保持应用状态不刷新页面**。

```javascript
// webpack.config.js
module.exports = {
  devServer: { hot: true },
  plugins: [
    // Webpack 5 无需手动添加 HmrPlugin，设置 devServer.hot 即可
  ],
};
```

```javascript
// 模块内手动处理 HMR
if (module.hot) {
  module.hot.accept('./renderer', () => {
    // 热更新时重新执行渲染
    render();
  });

  module.hot.dispose(() => {
    // 清理副作用：解绑事件、清理定时器
    clearInterval(timer);
  });
}
```

| 场景 | 是否支持 HMR |
| --- | --- |
| CSS（style-loader） | 支持，即时生效 |
| React（react-refresh） | 支持，组件状态保留 |
| Vue（vue-loader） | 支持 |
| 普通 JS 模块 | 需手动写 `module.hot.accept` |

!!! tip
    React 项目使用 `@pmmmwh/react-refresh-webpack-plugin` + `react-refresh` 即可获得接近 Vite 的组件级热更新体验。

## 代码分割 / Code Splitting

### splitChunks 分包

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',                      // 对同步/异步 chunk 都分包
      minSize: 20000,                     // 最小尺寸，小于此值不拆分
      cacheGroups: {
        // 第三方依赖单独打包
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: -10,
        },
        // 被多个入口引用的公共模块
        common: {
          name: 'common',
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

### 动态 import（异步 chunk）

```javascript
// 点击时才加载，产出一个独立 chunk
button.addEventListener('click', async () => {
  const { default: heavyModule } = await import('./heavy-module.js');
  heavyModule.run();
});

// React 路由懒加载
const Home = React.lazy(() => import('./pages/Home'));
```

### runtimeChunk

```javascript
module.exports = {
  optimization: {
    runtimeChunk: 'single',               // 抽出 webpack runtime，稳定长期缓存
  },
};
```

| 配置 | 作用 |
| --- | --- |
| `splitChunks.chunks: 'all'` | 全局分包 |
| `cacheGroups.vendors` | 第三方库独立 chunk |
| 动态 `import()` | 按需加载，减小首屏体积 |
| `runtimeChunk: 'single'` | runtime 独立，避免缓存失效 |

!!! tip
    动态 `import()` 是减小首屏体积最有效的手段；配合路由懒加载可显著缩短首屏时间。

## Tree Shaking 与 sideEffects

Tree Shaking 依赖 ES Module 的静态结构，删除未被引用的导出代码。

```json
// package.json：声明模块无副作用，让 webpack 放心摇树
{
  "sideEffects": false
}
```

```json
// 若有副作用（如全局样式、polyfill），用数组显式声明
{
  "sideEffects": ["*.css", "./src/polyfill.js"]
}
```

```javascript
// webpack.config.js
module.exports = {
  mode: 'production',                    // 生产模式才开启 Tree Shaking
  optimization: {
    usedExports: true,                   // 标记未使用导出
    providedExports: true,
    sideEffects: true,                   // 读取 package.json 的 sideEffects
  },
};
```

!!! warning
    以下写法**无法**被 Tree Shaking：CommonJS `require`、`import * as` 后动态访问属性、带副作用导入的 barrel 文件。优先使用具名导入 `import { fn } from 'pkg'`。

## 构建缓存 / Caching

```javascript
module.exports = {
  // Webpack 5 内置文件系统缓存
  cache: {
    type: 'filesystem',                  // 或 'memory'
    buildDependencies: { config: [__filename] },  // 配置变更时失效缓存
  },
  output: {
    // contenthash：内容变化才改变文件名，最大化利用浏览器缓存
    filename: 'js/[name].[contenthash:8].js',
    chunkFilename: 'js/[name].[contenthash:8].chunk.js',
    assetModuleFilename: 'assets/[hash][ext][query]',
  },
};
```

```javascript
// babel-loader 缓存
{ loader: 'babel-loader', options: { cacheDirectory: true } }
```

| 缓存对象 | 手段 |
| --- | --- |
| 构建过程 | `cache.type: 'filesystem'` |
| babel 转译结果 | `cacheDirectory: true` |
| 浏览器缓存 | `[contenthash]` 命名 + 长缓存 |
| 运行时 | `runtimeChunk: 'single'` |

!!! note
    `contenthash` 基于文件内容计算；`chunkhash` 基于 chunk；开发环境用 `hash` 即可。生产必须用 `[contenthash]`。

## 路径别名 / Path Alias

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      'react-dom': '@hot-loader/react-dom',   // 例：替换包
    },
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],  // 省略扩展名
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],  // 第三方包查找目录
  },
};
```

```json
// tsconfig.json：编辑器提示与 webpack 保持一致
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

!!! warning
    `resolve.alias` 与 `tsconfig.json` 的 `paths` 必须同步配置，否则会出现「能运行但编辑器报红」或反之。

## 性能优化汇总 / Performance

### 减小产物体积

| 手段 | 说明 |
| --- | --- |
| Tree Shaking | 删除未使用代码 |
| 代码分割 + 懒加载 | 首屏只加载必要 chunk |
| 压缩 | Terser（JS）+ css-minimizer（CSS） |
| 按需引入 | 组件库按需加载 |
| 移除重复依赖 | 用 `npm ls` 排查多版本 |
| `CompressionWebpackPlugin` | 产出 gzip / brotli |

### 提升构建速度

| 手段 | 说明 |
| --- | --- |
| `cache: filesystem` | 二次构建复用缓存 |
| `thread-loader` | 多线程执行 loader |
| `esbuild-loader` | 替代 babel-loader 转译 |
| `ForkTsCheckerWebpackPlugin` | 类型检查独立进程 |
| `include` / `exclude` | 缩小 loader 匹配范围 |
| DllPlugin / 外置 CDN | `externals` 外置大依赖 |

```javascript
// externals：把大依赖交给 CDN，不打包进产物
module.exports = {
  externals: {
    react: 'React',
    'react-dom': 'ReactDOM',
  },
};
```

!!! tip
    优化顺序建议：先「按需引入 + 代码分割」，再「压缩 + 缓存」，最后才考虑多线程/外置等提速手段。

## Webpack 与 Vite 对比 / Webpack vs Vite

| 对比项 | Webpack | Vite |
| --- | --- | --- |
| 开发启动 | 需先打包，较慢 | 基于原生 ESM，按需编译，极快 |
| HMR 速度 | 随项目增大变慢 | 与模块数弱相关，极快 |
| 生产构建 | 自研打包器 | Rollup |
| 配置复杂度 | 高，loader/plugin 组合灵活 | 低，约定优于配置 |
| 生态 | 极其成熟，历史插件海量 | 快速增长 |
| 兼容性 | 极强，可兼容旧浏览器 | 现代浏览器，旧浏览器需 legacy 插件 |
| 适用场景 | 复杂/存量/legacy 项目、深度定制 | 新项目、现代浏览器、快速迭代 |

```bash
# 选型建议
# 新项目 → Vite；存量复杂项目或需深度定制打包 → Webpack
```

## 常见问题 / FAQ

| 问题 | 原因 | 解决 |
| --- | --- | --- |
| 刷新页面 404 | SPA 路由未兜底 | `devServer.historyApiFallback: true` 或 Nginx `try_files` |
| CSS 未生效 | loader 顺序错误 | 调整顺序：`sass → postcss → css → style/MiniCss` |
| 环境变量为 undefined | `process.env` 未注入 | 用 `DefinePlugin` 注入 |
| 别名编辑器报红 | 只配了 webpack | 同步 `tsconfig.json` 的 `paths` |
| 构建产物过大 | 重复依赖 / 未按需 | 查 `BundleAnalyzerPlugin`，做分包与按需 |
| 二次构建慢 | 未开缓存 | 配置 `cache.filesystem` 与 `cacheDirectory` |

```nginx
# 生产部署：SPA 兜底 + gzip_static
location / {
  try_files $uri $uri/ /index.html;
}
location ~* \.(js|css|png|jpg|svg|woff2)$ {
  expires 1y;
  add_header Cache-Control "public, immutable";
  gzip_static on;
}
```

```json
// package.json 环境变量注入示例
{
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --config config/webpack.prod.js"
  }
}
```

!!! warning
    生产环境务必使用 `[contenthash]` 命名并由 Nginx 设置长缓存；`index.html` 不可强缓存（需 `no-cache`），否则发版后用户拿到旧入口。

!!! tip
    - 存量项目迁移不必一步到位：先开 `cache.filesystem` 提速，再逐步做分包与按需。
    - 遇到 loader/plugin 报错，优先用 `--stats detailed` 查看模块依赖与耗时分布。
