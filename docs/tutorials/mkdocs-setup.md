---
title: MkDocs 搭建教程
tags:
  - 教程
  - MkDocs
  - Material
---

# MkDocs 搭建教程

> 从零搭建一个 Markdown 技术文档站：安装、配置 `mkdocs.yml`、编写内容、常用命令、部署与排障。带 `< >` 的部分需替换为实际值，`[ ]` 表示可选参数。

!!! note 
    **⚠️ 版本提醒（2026-10）**：MkDocs 当前最新稳定版为 **1.6.1**；1.6 之后是推倒重写的 2.0，目前仅有预发布版（2.0.devX），与现有主题、插件生态**不兼容**，安装时务必锁定 `mkdocs~=1.6`。最流行的 Material 主题（mkdocs-material 9.7.7）已进入维护模式，官方计划 2026-11-05 起停止功能更新；新项目可继续放心使用 9.7.x，也可关注其分支 MaterialX 或原团队的替代品 Zensical。

  
---

## 一、前置条件与环境准备

| 依赖 | 要求 | 检查命令 |
|---|---|---|
| Python | ≥ 3.8（建议 3.11+） | `python --version` |
| 包管理器 | pip 或 uv（二选一） | `pip --version` / `uv --version` |

强烈建议为每个文档项目使用独立虚拟环境，避免污染全局：

```powershell
# 方式 A：venv + pip（标准做法）
python -m venv .venv
.\.venv\Scripts\Activate.ps1        # Windows PowerShell；Linux/macOS 用 source .venv/bin/activate

# 方式 B：uv（更快，本套笔记有 uv 专篇）
uv venv
.\.venv\Scripts\Activate.ps1
```

---

## 二、安装 MkDocs 与 Material 主题

```bash
# 锁定 1.6 稳定线，防止误装 2.0 预发布版
pip install "mkdocs~=1.6.1" "mkdocs-material~=9.7.7"
# 或
uv pip install "mkdocs~=1.6.1" "mkdocs-material~=9.7.7"

# 验证安装
mkdocs version
pip show mkdocs mkdocs-material
```

安装后把依赖固化到文件，方便 CI 和其他人复现：

```bash
pip freeze > requirements.txt        # uv 项目用 uv lock / uv export
```

---

## 三、创建项目与目录结构

```bash
mkdocs new my-docs                 # 新建项目骨架
cd my-docs
tree /F                            # Windows 查看结构（Linux 用 tree）
```

生成的标准结构：

```
my-docs/
├── mkdocs.yml        # 唯一配置文件（YAML）
└── docs/
    └── index.md      # 站点首页
```

推荐扩展后的完整结构：

```
docs/
├── index.md            # 首页
├── guide/              # 按板块分目录
│   ├── install.md
│   └── usage.md
├── reference/
│   └── api.md
├── assets/             # 图片、favicon 等静态资源
│   └── img/
└── stylesheets/
    └── extra.css       # 自定义样式
```

> 注意：`mkdocs.yml` 中未写进 `nav` 的 md 文件也会参与构建（可通过 URL 访问），但不会出现在导航中；`docs/` 下的非 md 文件按原样复制到站点根。

---

## 四、mkdocs.yml 核心配置

最小可用配置：

```yaml
site_name: 我的文档站
site_description: 一句话介绍
theme:
  name: material
```

生产级常用配置模板：

```yaml
site_name: 项目文档
site_url: https://example.com/docs/          # 部署地址，影响 sitemap
repo_url: https://github.com/<user>/<repo>
repo_name: <user>/<repo>

theme:
  name: material
  language: zh                                # 界面中文
  favicon: assets/img/favicon.png
  logo: assets/img/logo.png
  features:
    - navigation.sections                     # 一级导航分组展开
    - navigation.top                          # 返回顶部按钮
    - navigation.tracking                     # 地址栏跟随章节
    - search.suggest
    - content.code.copy                       # 代码块一键复制
  palette:
    - scheme: default                         # 亮色
      primary: indigo
      toggle: {icon: material/weather-sunny, name: 切换暗色}
    - scheme: slate                           # 暗色
      accent: indigo
      toggle: {icon: material/weather-night, name: 切换亮色}

# 内置 markdown_extensions（MkDocs 自带）+ PyMdown 扩展（随 material 安装）
markdown_extensions:
  - admonition                                # 提示块 !!! note
  - attr_list
  - toc:
      permalink: true                         # 标题锚点
  - pymdownx.highlight:
      anchor_line_numbers: false
      use_pygments: true
  - pymdownx.superfences:                     # 嵌套代码块 / mermaid
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.tabbed:
      alternate_style: true                   # 标签页
  - pymdownx.tasklist:
      custom_checkbox: true                   # 任务清单
  - footnotes

plugins:
  - search:
      lang:
        - zh                                  # 中文全文搜索
        - en

nav:
  - 首页: index.md
  - 入门:
      - 安装: guide/install.md
      - 使用: guide/usage.md
  - 参考:
      - API: reference/api.md

extra_css:
  - stylesheets/extra.css

extra:
  social:
    - icon: fontawesome/brands/github
      link: https://github.com/<user>
```

> YAML 常见坑：缩进只能用空格；`!!python/name:` 标签要求文件以 YAML 1.1 解析（MkDocs 默认支持）；`nav` 里路径相对 `docs/`。

---

## 五、编写内容

### 1. 基本 Markdown

标准 CommonMark 即可：标题、列表、表格、链接、图片。站内链接用**相对 md 路径**，构建时自动转成正确的 html 链接：

```markdown
[安装说明](guide/install.md)
![架构图](../assets/img/arch.png)
```

### 2. 提示块（Admonition）

```markdown
!!! note "标题可省略"
    普通提示内容。

!!! warning
    警告内容。

!!! tip
    小技巧。
```

支持类型：`note / abstract / info / tip / success / question / warning / failure / danger / bug`，前缀加 `!!!+` 默认展开、`!!!-` 默认折叠。

### 3. 代码高亮与标签页

````markdown
```python title="demo.py"
print("hello")
```

=== "Windows"
    ```powershell
    pip install mkdocs
    ```

=== "Linux"
    ```bash
    pip3 install mkdocs
    ```
````

### 4. Mermaid 图表

上面 `mkdocs.yml` 已配置 `superfences.custom_fences`，直接写：

````markdown
```mermaid
graph LR
  A[写 Markdown] --> B[mkdocs build] --> C[静态站点]
```
````

### 5. 表格对齐、任务清单、脚注

```markdown
| 左对齐 | 居中 | 右对齐 |
|:-------|:----:|-------:|
| a      | b    | c      |

- [x] 已完成
- [ ] 未完成

带脚注的文本[^1]

[^1]: 脚注内容。
```

---

## 六、常用命令速查

| 命令 | 说明 |
|---|---|
| `mkdocs new <目录>` | 新建项目骨架 |
| `mkdocs new -r` | 生成 `readthedocs.yml` |
| `mkdocs serve` | 本地预览，默认 <http://127.0.0.1:8000>，保存即热更新 |
| `mkdocs serve -a 0.0.0.0:8000` | 监听所有网卡（局域网访问） |
| `mkdocs serve --dev-addr 127.0.0.1:9000` | 换端口 |
| `mkdocs serve --dirty` | 只重建改动的页面（大站提速） |
| `mkdocs build` | 输出静态站点到 `site/` |
| `mkdocs build --clean` | 先清空 `site/` 再构建（CI 推荐） |
| `mkdocs build --strict` | 警告即报错：死链、缺 nav 立即失败（**CI 必开**） |
| `mkdocs build -f <配置>` | 指定配置文件 |
| `mkdocs gh-deploy` | 一键构建并发布到 GitHub Pages |
| `mkdocs gh-deploy --force` | 覆盖已存在远端分支 |
| `mkdocs gh-deploy --remote-name <remote>` | 推到自定义 remote |
| `mkdocs serve -v` | 详细日志排查构建问题 |
| `mkdocs new .` / 其他 `--help` | 查看子命令完整参数 |

典型工作流：

```bash
mkdocs new my-docs && cd my-docs
pip install "mkdocs~=1.6.1" "mkdocs-material~=9.7.7"
mkdocs serve            # 边写边看
# 写完提交
mkdocs build --strict   # 本地预检死链
mkdocs gh-deploy        # 发布
```

---

## 七、部署方式

### 1. GitHub Pages（最简单）

```bash
mkdocs gh-deploy
```

自动推送到 `gh-pages` 分支。站点在仓库 Settings → Pages 中选择 `gh-pages` 分支后生效，地址形如 `https://<user>.github.io/<repo>/`。私有仓库需在 Settings → Pages 开启权限。首次部署建议加 `--force`。

CI 自动发布（`.github/workflows/docs.yml` 核心步骤）：

```yaml
- uses: actions/setup-python@v5
  with: {python-version: "3.12"}
- run: pip install -r requirements.txt
- run: mkdocs build --strict
- run: mkdocs gh-deploy --force
```

### 2. GitLab Pages

`.gitlab-ci.yml`：

```yaml
pages:
  image: python:3.12-slim
  script:
    - pip install -r requirements.txt
    - mkdocs build --clean --site-dir public
  artifacts:
    paths: [public]
```

### 3. Nginx / 任意静态服务器

`site/` 就是普通静态文件，scp/rsync 上传后指向根目录即可：

```nginx
server {
    listen 80;
    server_name docs.example.com;
    root /var/www/my-docs/site;
}
```

### 4. 子路径部署

站点放在 `https://example.com/docs/` 下时，配置 `site_url: https://example.com/docs/`，构建产物整体拷入对应目录即可，资源路径会自动带上子路径。

---

## 八、进阶：多版本与中英双语

- **多版本**：原生 1.x 需借助 `mike`（针对 mkdocs-material 的 `version` provider）或 `mkdocs-multirepo-plugin`；思路是每个版本各 build 一份，导航页聚合。
  ```bash
  pip install mike
  mike deploy --push --update-aliases v1.0.0 v1 latest
  ```
- **双语**：常用 `mkdocs-static-i18n` 插件，按 `docs/zh/`、`docs/en/` 目录自动翻译互链。
- **API 文档**：`mkdocstrings[python]` 插件直接从 Python docstring 生成参考页：
  ```yaml
  plugins:
    - mkdocstrings:
        handlers:
          python:
            options: {docstring_style: google}
  ```
  正文中写 `::: mypackage.module` 即可嵌入。

---

## 九、常见故障对照

| 现象 | 原因与处理 |
|---|---|
| 装完后 `mkdocs` 命令找不到 | 虚拟环境未激活，或 Scripts 目录不在 PATH；重新 `activate` |
| 构建正常但页面全 404 | `nav` 里写了不存在的文件；`mkdocs build --strict` 会直接报出，按提示修路径 |
| 图片不显示 | 路径要相对当前 md 文件（或统一从 `docs/` 根写绝对路径 `/assets/img/x.png` 并配合 `use_directory_urls: false`） |
| 中文搜索无结果 | `plugins.search.lang` 未配 `zh` |
| 升级后主题报错 | 误升到了 MkDocs 2.0 预发布版；卸载后重装 `pip install "mkdocs~=1.6.1"` |
| 本地正常、Pages 空白 | 仓库未开启 Pages 分支，或推的是旧 `gh-pages`；`--force` 重推并检查 Settings → Pages |
| Windows 控制台乱码/报错 | 终端编码非 UTF-8，`chcp 65001` 或改用 Windows Terminal |
| 大项目 build 很慢 | `serve --dirty`；减少 `--strict` 下无用的孤儿页告警，或把图片资源放 CDN |

---

## 十、高频场景速查

| 我要… | 命令 / 配置 |
|---|---|
| 快速起一个文档站 | `mkdocs new d && cd d && pip install mkdocs-material && mkdocs serve` |
| 加中文搜索 | `plugins: - search: {lang: [zh, en]}` |
| 加 mermaid 图 | `pymdownx.superfences.custom_fences`（见第四节模板） |
| 发布 GitHub Pages | `mkdocs gh-deploy` |
| CI 防死链 | `mkdocs build --strict` |
| 换端口预览 | `mkdocs serve --dev-addr 127.0.0.1:9000` |
| 局域网给同事看 | `mkdocs serve -a 0.0.0.0:8000` |
| 一键部署到远端 | 将 `site/` 上传任意静态服务器 / OSS / Netlify |
| Python 项目 API 文档 | 安装 `mkdocstrings`，写 `::: module` |
| 锁定版本防生态变动 | `requirements.txt` 固定 `mkdocs==1.6.1`、`mkdocs-material==9.7.7` |

---

## 十一、使用建议

1. **务必锁定版本**：MkDocs 2.0 重写版与 1.x 插件生态不兼容，且当前 PyPI 上游维护状态动荡（Material 主题 2026-11 起停止功能更新）；`requirements.txt` 固定可用版本，CI 中同理。
2. **CI 永远开 `--strict`**：死链和缺 nav 的警告在上线前暴露，而不是让用户 404。
3. **内容与技术分离**：md 里不要写内联 HTML 和绝对域名链接，方便以后换主题、换部署地址。
4. **资源统一目录**：图片放 `docs/assets/`，站内引用相对路径，随站点一起构建，避免外链失效。
5. **备份配置**：`mkdocs.yml` 与 `requirements.txt` 一起进 Git，随时可在新机复现整站。

---

*参考：MkDocs 官方文档 <https://www.mkdocs.org/> · Material for MkDocs <https://squidfunk.github.io/mkdocs-material/> · 插件目录 <https://github.com/mkdocs/catalog>*

