---
title: uv 速查
tags:
  - 工具
  - uv
---
# uv 常用命令使用说明

> uv 是由 Astral（Ruff 团队）用 Rust 编写的极快 Python 包管理与项目运行工具，可替代 pip、pip-tools、virtualenv、pyenv、pipx 的大部分功能。带 `< >` 的部分需替换为实际值，`[ ]` 表示可选参数。

---

## 一、安装与自检

```powershell
# Windows（PowerShell）安装
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

```bash
# macOS / Linux 安装
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或通过 pip 安装
pip install uv
```

```bash
uv --version          # 查看版本
uv help               # 查看全部命令分组
uv self update        # 升级 uv 自身
uv self uninstall     # 卸载 uv
```

---

## 二、项目管理（推荐工作流）

uv 的项目模式围绕 `pyproject.toml` + `uv.lock` + `.venv` 三件套。

```bash
uv init [项目名]           # 初始化新项目（生成 pyproject.toml、main.py 等）
uv init --library [项目名] # 初始化库/包项目（src 布局）
uv add <包名>              # 添加依赖并自动同步环境（如 uv add requests）
uv add --dev <包名>        # 添加开发依赖（pytest、ruff 等）
uv add "<包>[extra]"       # 带可选功能，如 uv add "fastapi[standard]"
uv remove <包名>           # 删除依赖
uv sync                    # 按 uv.lock 把 .venv 恢复到与锁文件一致
uv sync --extra <功能名>   # 同步并包含 pyproject 中定义的 optional 依赖组
uv lock                    # 生成/更新 uv.lock
uv lock --upgrade          # 升级所有依赖到允许范围内最新并更新锁
uv lock --upgrade-package <包名>  # 只升级某个依赖
uv tree                    # 查看当前项目依赖树
uv run <命令>              # 在项目环境中运行命令（自动先 sync），如 uv run python main.py / uv run pytest
uv run --no-sync <命令>    # 跳过自动同步，直接运行
uv run --with <临时包> <命令>  # 临时注入某个包运行，不写入项目依赖
```

> 提示：项目下用 `uv run` 代替 `python`，无需手动激活虚拟环境；克隆别人的 uv 项目后先 `uv sync`。

---

## 三、虚拟环境（venv）

```bash
uv venv [路径]                 # 在当前目录创建 .venv（默认名 .venv）
uv venv --python 3.12 [路径]   # 指定 Python 版本创建
uv venv --seed                 # 创建时预装 pip/setuptools
source .venv/bin/activate      # Linux/macOS 激活
.venv\Scripts\Activate.ps1     # Windows PowerShell 激活
deactivate                     # 退出虚拟环境
```

---

## 四、Python 版本管理（替代 pyenv）

```bash
uv python install 3.12         # 下载安装指定版本 Python
uv python install              # 安装 pyproject 中要求的版本
uv python list                 # 列出可安装/已安装的 Python
uv python find                 # 查看当前 uv 会选用哪个 Python
uv python pin 3.12             # 在项目里固定版本（写入 .python-version）
uv python upgrade              # 升级托管的 Python 补丁版本
uv python uninstall 3.11       # 卸载指定版本
```

---

## 五、pip 兼容层（脚本/已有环境场景）

不建项目、只装包时可以直接把 uv 当"加速版 pip"用：

```bash
uv pip install <包名>                  # 安装包（等价 pip install）
uv pip install -r requirements.txt     # 按 requirements 安装
uv pip install -e .                    # 可编辑安装当前项目
uv pip uninstall <包名>                # 卸载
uv pip list                            # 列出已安装包
uv pip freeze                          # 导出精确版本清单
uv pip show <包名>                     # 查看包详情
uv pip check                           # 检查依赖是否冲突
uv pip compile requirements.in -o requirements.txt  # 由高层约束生成锁定清单（pip-tools 式）
uv pip sync requirements.txt           # 让环境严格等于清单（多退少补）

# 常用全局参数
--python <解释器路径>    # 指定装到哪个环境
--system                 # 直接装进系统 Python（不推荐）
--dry-run                # 预览将执行的操作，不实际安装
```

> 注意：`uv pip` 系列命令需要先激活虚拟环境，或用 `--python` 显式指定目标环境，否则会提示找不到环境。

---

## 六、运行与安装命令行工具（替代 pipx）

```bash
uvx <工具名>                # 临时运行一个 CLI 工具，用完即走（等价 uv tool run）
uvx ruff check .            # 示例：不安装直接跑 ruff
uvx "black[jupyter]" .      # 带 extras

uv tool install <工具名>    # 持久安装 CLI 工具（隔离环境，全局可执行）
uv tool install --python 3.12 <工具名>  # 指定运行版本
uv tool list                # 查看已安装的工具
uv tool upgrade <工具名>    # 升级某个工具
uv tool run <工具名>        # 运行已安装的工具（同 uvx）
uv tool uninstall <工具名>  # 卸载
uv tool update-shell        # 把工具目录加入 PATH
```

---

## 七、脚本与依赖内联

```bash
uv run script.py            # 直接运行单文件脚本
# 在 .py 顶部写内联依赖（PEP 723），uv 会自动装好再跑：
# /// script
# dependencies = ["requests", "rich"]
# ///
```

---

## 八、构建与发布（打包者）

```bash
uv build                     # 构建 sdist + wheel 到 dist/
uv build --wheel             # 只构建 wheel
uv publish                   # 发布到 PyPI（需 UV_PUBLISH_TOKEN 或 ~/.pypirc）
uv publish --token <pypi-token>
```

---

## 九、缓存管理

```bash
uv cache clean               # 清空全部缓存
uv cache clean <包名>        # 只清某个包的缓存
uv cache dir                 # 查看缓存目录位置
uv cache info                # 查看缓存占用统计
```

---

## 十、高频场景速查

| 场景 | 操作 |
|---|---|
| 从零开始一个新 Python 项目 | `uv init myproj` → `cd myproj` → `uv add fastapi` → `uv run python main.py` |
| 克隆了 uv 项目，环境怎么恢复 | `uv sync`，然后用 `uv run ...` 执行 |
| 老项目只有 requirements.txt | `uv venv --python 3.12` → `uv pip install -r requirements.txt` |
| 只想临时用一下某个命令行工具 | `uvx <工具>`，不必安装 |
| 把项目 Python 固定在 3.11 | `uv python pin 3.11` |
| 升级单个依赖 | `uv lock --upgrade-package <包名>` → `uv sync` |
| 导出当前环境精确版本清单 | `uv pip freeze > requirements.txt`（项目模式也可 `uv export`） |
| CI 里追求可复现安装 | `uv sync --frozen`（严格按锁文件，不更新） |
| 装依赖报错提示版本冲突 | 看 `uv tree` 定位冲突来源，调整版本约束后 `uv lock` |
| uv 装包慢/想换镜像源 | 在项目 `pyproject.toml` 或全局配置中设置 `[[tool.uv.index]]` 国内镜像地址 |

---

## 十一、建议与提醒

1. **优先用项目模式**：新项目一律 `uv init` + `uv add`，让 `uv.lock` 进版本库，保证团队环境可复现。
2. **`uv run` 是日常主入口**：跑脚本、跑测试、跑构建都走它，减少"环境没激活"类问题。
3. **`uv pip` 只作过渡**：迁移旧项目或写脚本时好用，但长期维护的项目建议尽快转 `pyproject.toml` 工作流。
4. **工具类用 `uvx`/`uv tool`**：不要再用 `pip install` 把 black、ruff 一类工具装进项目环境。
5. **锁文件冲突处理**：`uv.lock` 出现合并冲突时不要手工挑行，直接 `uv lock` 重新生成。

---

