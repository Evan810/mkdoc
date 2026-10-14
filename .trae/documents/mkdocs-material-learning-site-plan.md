# MkDocs-Material 个人学习网站实施规划

## Context

用户希望搭建一个基于 **mkdocs-material** 的个人学习记录网站，风格参考 `docs.bugdesigner.cn`。当前项目已初始化 venv 并安装了 `mkdocs-material>=9.7.7`，但 `mkdocs.yml` 仅有最简配置，无目录结构和插件。本规划将完成从基础配置到完整站点的全部搭建。

---

## 当前状态

| 项目 | 状态 |
|------|------|
| `mkdocs.yml` | 最简配置（site_name: My Docs, theme: material） |
| `docs/index.md` | 默认欢迎页 |
| `pyproject.toml` | 仅有 mkdocs-material 依赖 |
| 目录结构 | 无分类目录、无 blog、无 tags |
| `uv.lock` | 存在（依赖已锁定） |

---

## 实施计划

### Step 1: 安装新增依赖

**文件**: `pyproject.toml` + 运行 `uv add`

需要新增：
- `pymdown-extensions>=10.0` — PyMdownx 系列 Markdown 扩展（material 风格必备）
- `mkdocs-rss-plugin>=1.17` — RSS 订阅

> 注：`git-revision-date-localized` 作为可选项，等确定有 git 仓库后再安装。Pygments 通常已随 mkdocs-material 安装。

---

### Step 2: 完整配置 mkdocs.yml

**文件**: `mkdocs.yml`

配置以下模块（完整内容见附录 A）：

| 模块 | 说明 |
|------|------|
| site_name / site_description / site_url | 用户需替换为自己的信息 |
| repo_url / repo_name | 需填入实际 GitHub 仓库地址（可选） |
| theme.features | 启用约 25 个特性（导航、搜索、TOC、代码块等） |
| theme.palette | 亮色/暗色双主题，indigo 主色 |
| markdown_extensions | admonition、pymdownx 全套、toc permalink 等 |
| plugins | blog → tags → search → social → rss |
| nav | 按分类组织的完整导航（参考 bugdesigner.cn 分类方式） |
| extra.social | GitHub 等社交链接 |

**导航分类**（参考 bugdesigner.cn + 通用学习需求）：
1. 首页
2. 前端开发（React / Vue / TypeScript / 工程化）
3. 后端开发（SpringBoot / Node.js）
4. Python（基础 / 爬虫 / Django / 数据分析）
5. 网络安全（信息收集 / Web 攻防 / 内网渗透 / 漏洞复现）
6. 数学建模（数据分析 / 评价 / 预测 / 优化 / 分类）
7. 实战项目
8. 教程与工具
9. Blog（自动）
10. Tags（自动）

> **用户可按需裁剪分类**，这是参考结构，不是硬性要求。

---

### Step 3: 创建目录结构

按以下目录树创建（仅列顶层结构）：

```
docs/
├── index.md                    # 首页
├── tags.md                     # 标签索引
├── blog/
│   ├── .authors.yml            # 作者信息
│   └── posts/                  # 博客文章
├── assets/
│   ├── images/
│   ├── stylesheets/extra.css   # 自定义样式
│   └── javascripts/
├── includes/
│   └── abbreviations.md        # 缩写定义
├── front-end/                  # 前端分类
├── back-end/                   # 后端分类
├── python/                     # Python 分类
├── net-sec/                    # 网络安全
├── math-model/                 # 数学建模
├── projects/                   # 实战项目
├── tutorials/                   # 教程
└── tools/                      # 工具

overrides/
└── main.html                   # 模板覆盖入口（注入 extra.css/js）
```

每个分类子目录内含一个 `index.md`（分类首页）。

---

### Step 4: 创建首页 docs/index.md

采用 **纯 Markdown + 内置 grid cards** 方案：
- Hero 区域（标题 + 一句话描述）
- 关于我（简短个人介绍）
- 技能树（使用 mermaid mindmap）
- 分类导航卡片（8 个 grid cards，每个含图标 + 描述 + 链接）
- 最新博客列表

---

### Step 5: 配置 blog 插件

- 创建 `docs/blog/.authors.yml`（作者信息）
- 创建示例博客文章 `docs/blog/posts/2026-10-14-hello-world.md`
- 博客自动生成索引、分类页、标签页

---

### Step 6: 创建自定义样式

**文件**: `docs/assets/stylesheets/extra.css` + `overrides/main.html`

`overrides/main.html` 继承 `base.html`，在 `extrahead` block 注入 extra.css。

extra.css 包含：
- 卡片网格美化（圆角、hover 效果）
- 代码块内边距微调
- 标题锚点悬停显示
- Hero 区域样式
- 暗色模式适配

---

### Step 7: 创建分类索引页

为每个分类目录创建 `index.md`（简单的分类介绍 + 子页面链接）。

---

### Step 8: 本地验证

运行 `uv run mkdocs serve` 启动开发服务器，检查：
- 页面能否正常加载
- 导航是否正确展开
- 搜索是否可用
- 博客是否正常显示
- 暗色模式切换是否生效

运行 `uv run mkdocs build --strict` 检查构建是否有警告/错误。

---

### Step 9: 部署（可选）

如果用户需要部署，创建 `.github/workflows/deploy.yml`：
- 使用 `actions/checkout@v4` + `fetch-depth: 0`
- 使用 `astral-sh/setup-uv@v3`
- 构建 + 上传到 GitHub Pages

---

## 关键技术决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 首页方案 | Markdown + grid cards | 简单、无需写 HTML 模板、material 内置支持 |
| 博客功能 | 内置 blog 插件 | mkdocs-material 9.2+ 内置，无需额外安装 |
| 标签功能 | 内置 tags 插件 | mkdocs-material 8.2+ 内置 |
| 代码高亮 | Pygments | mkdocs-material 默认，配合 pymdownx.highlight |
| 数学公式 | pymdownx.arithmatex | 内置支持，无需 KaTeX 外部依赖 |
| 部署平台 | GitHub Pages（推荐） | 免费、与 git 集成好 |
| 暗黑模式 | palette media 查询 | 跟随系统偏好 + 手动切换 |

---

## 需要用户提供的信息

实施时需要填入以下个性化信息（占位符已标注）：

1. **site_name**: 网站名称（当前是 "My Docs"）
2. **site_description**: 一句话描述
3. **site_url**: 实际域名（如 `https://username.github.io/repo/`）
4. **repo_url**: GitHub 仓库地址（可选）
5. **author**: 作者名
6. **分类结构**: 是否需要调整上面的 8 个分类

---

## 验证步骤

```bash
# 1. 安装依赖
uv add pymdown-extensions mkdocs-rss-plugin

# 2. 本地预览
uv run mkdocs serve
# 打开 http://127.0.0.1:8000 检查

# 3. 构建检查
uv run mkdocs build --strict
# 无错误输出 = 通过
```

---

## 附录 A: 完整 mkdocs.yml 配置

（实施时直接使用以下配置，替换占位符）

```yaml
# ============================================================
# 基础信息
# ============================================================
site_name: 我的学习笔记
site_description: 个人学习记录，涵盖前端、后端、Python、网络安全、数据建模等
site_url: https://your-username.github.io/your-repo/
author: Your Name

# 仓库链接（可选）
repo_url: https://github.com/your-username/your-repo
repo_name: GitHub
edit_uri: edit/main/docs/

copyright: Copyright &copy; 2025-2026 Your Name

# ============================================================
# 主题配置
# ============================================================
theme:
  name: material
  language: zh-CN
  font:
    text: Inter
    code: JetBrains Mono

  icon:
    logo: material/book-open-page-variant
    favicon: material/book-open-page-variant
    repo: fontawesome/brands/github

  features:
    # 导航
    - navigation.instant
    - navigation.instant.prefetch
    - navigation.instant.progress
    - navigation.tracking
    - navigation.expand
    - navigation.path
    - navigation.indexes
    - navigation.tabs
    - navigation.tabs.sticky
    - navigation.sections
    - navigation.top
    - navigation.footer
    # 搜索
    - search.highlight
    - search.suggestions
    - search.share
    # TOC
    - toc.follow
    - toc.sticky
    # 内容
    - content.code.copy
    - content.code.annotate
    - content.tabs.link
    - content.tooltips
    - content.footnote.tooltips

  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: 切换暗色模式
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: 切换亮色模式
    - scheme: default
      toggle:
        icon: material/brightness-7
        name: 切换暗色模式
    - scheme: slate
      toggle:
        icon: material/brightness-4
        name: 切换亮色模式

  custom_dir: overrides

# ============================================================
# Markdown 扩展
# ============================================================
markdown_extensions:
  - abbr
  - admonition
  - attr_list
  - def_list
  - footnotes
  - md_in_html
  - tables
  - toc:
      permalink: true
      permalink_title: "Permalink to this heading"
      toc_depth: 4
  - pymdownx.arithmatex:
      generic: true
  - pymdownx.betterem:
      smart_enable: all
  - pymdownx.caret
  - pymdownx.details
  - pymdownx.emoji:
      emoji_index: !!python/name:material.extensions.emoji.twemoji
      emoji_generator: !!python/name:material.extensions.emoji.to_svg
  - pymdownx.highlight:
      anchor_linenums: true
      line_spans: __span
      pygments_lang_class: true
  - pymdownx.inlinehilite
  - pymdownx.keys
  - pymdownx.mark
  - pymdownx.smartsymbols
  - pymdownx.snippets
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.tasklist:
      custom_checkbox: true
  - pymdownx.tilde

# ============================================================
# 插件
# ============================================================
plugins:
  - blog:
      blog_dir: blog
      post_dir: "{blog}/posts"
      categories: true
      pagination: true
      pagination_per_page: 10
  - tags:
      tags_file: tags.md
  - search
  - social
  - rss:
      feed_ttl: 1440

# ============================================================
# 导航
# ============================================================
nav:
  - 首页: index.md
  - 前端开发:
      - React: front-end/react.md
      - Vue: front-end/vue.md
      - TypeScript: front-end/typescript.md
      - 工程化:
          - Vite: front-end/engineering/vite.md
          - Webpack: front-end/engineering/webpack.md
  - 后端开发:
      - SpringBoot:
          - 集成技术: back-end/springboot/integrate-tech.md
          - RabbitMQ: back-end/springboot/rabbitmq.md
          - ElasticSearch: back-end/springboot/elasticsearch.md
          - MyBatis-Plus: back-end/springboot/mybatis-plus.md
          - 项目模板: back-end/springboot/springboot-template.md
      - Node.js:
          - Express: back-end/nodejs/express.md
          - NestJS: back-end/nodejs/nestjs.md
  - Python:
      - 基础语法: python/basic.md
      - 爬虫: python/spider.md
      - Django: python/django.md
      - 数据分析: python/data-analysis.md
  - 网络安全:
      - 信息收集: net-sec/info-collect.md
      - Web 攻防: net-sec/web-aad.md
      - 内网渗透: net-sec/intranet-penetration.md
      - 漏洞复现: net-sec/vuln-repro.md
  - 数学建模:
      - 数据分析: math-model/data-analysis.md
      - 评价类模型: math-model/evaluation.md
      - 预测类模型: math-model/forecast.md
      - 优化类模型: math-model/optimization.md
      - 分类模型: math-model/classification.md
  - 实战项目:
      - B站爬虫: projects/bilibili-spider.md
      - 练通爬取: projects/dlt-spider.md
  - 教程与工具:
      - 教程:
          - Electron 打包: tutorials/electron-package.md
          - MkDocs 搭建: tutorials/mkdocs-setup.md
      - 工具:
          - Git: tools/git.md
          - Docker: tools/docker.md
          - Linux: tools/linux.md
          - 图标库: tools/icons.md
          - 小知识: tools/tips.md
  - Blog: blog/index.md
  - Tags: tags.md

# ============================================================
# 额外配置
# ============================================================
extra:
  social:
    - icon: fontawesome/brands/github
      link: https://github.com/your-username
      name: GitHub
```
