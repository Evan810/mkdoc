---
title: Git 速查
tags:
  - 工具
  - Git
  - 版本控制
---

## 介绍 / Introduction

Git 常用命令速查，覆盖日常提交、分支管理、远程同步、撤销操作等高频场景。

> 整理自 Git 日常开发常用操作，覆盖从初始化到远程协作的完整流程。带 `< >` 的部分需替换为实际值，`[ ]` 表示可选参数。

---

## 一、初始化与配置

| 命令 | 说明 |
|---|---|
| `git init` | 在当前目录初始化一个新的 Git 仓库 |
| `git clone <url>` | 克隆远程仓库到本地 |
| `git clone <url> <目录名>` | 克隆并指定本地目录名 |
| `git config --global user.name "<姓名>"` | 设置全局提交用户名 |
| `git config --global user.email "<邮箱>"` | 设置全局提交邮箱 |
| `git config --list` | 查看当前所有 Git 配置 |
| `git config --global alias.st status` | 设置别名（之后可用 `git st`） |
| `git config --global core.autocrlf input` | 换行符设置（macOS/Linux 建议 input，Windows 建议 true） |

---

## 二、基础工作流（增删改查）

### 查看状态

```bash
git status              # 查看工作区与暂存区状态
git status -s           # 简洁模式输出
git diff                # 查看未暂存的修改
git diff --staged       # 查看已暂存、待提交的修改
git diff <分支A> <分支B> # 对比两个分支差异
```

### 跟踪与暂存

```bash
git add <文件>          # 将指定文件加入暂存区
git add .               # 将当前目录所有变更加入暂存区
git add -p              # 交互式逐块选择要暂存的修改
```

### 提交

```bash
git commit -m "<提交说明>"       # 提交暂存区内容
git commit -am "<说明>"          # 跳过 add，直接提交已跟踪文件的修改
git commit --amend -m "<新说明>" # 修改上一次提交（未推送前使用）
```

### 删除与移动

```bash
git rm <文件>           # 从仓库删除文件（同时删除工作区文件）
git rm --cached <文件>  # 仅取消跟踪，保留本地文件（常用于误提交的临时文件）
git mv <旧名> <新名>    # 重命名/移动文件
```

---

## 三、撤销与恢复

```bash
git checkout -- <文件>        # 丢弃工作区中该文件的修改（旧版 Git）
git restore <文件>            # 同上（新版 Git 推荐）
git restore --staged <文件>   # 把已暂存的文件移出暂存区，保留修改
git reset HEAD <文件>         # 取消暂存（旧版写法）

git reset --soft <commit>     # 回退到指定提交，保留修改在暂存区
git reset --mixed <commit>    # 回退并取消暂存，保留工作区修改（默认）
git reset --hard <commit>     # 回退并丢弃所有修改（危险，慎用）

git revert <commit>           # 生成一次新提交来抵消指定提交（已推送时推荐）
git clean -fd                 # 清理未被跟踪的文件和目录（危险，先 git clean -nd 预览）
git stash                     # 临时保存当前未提交的修改
git stash pop                 # 恢复最近一次 stash 并删除记录
git stash list                # 查看所有 stash 记录
```

---

## 四、分支管理

```bash
git branch                      # 查看本地分支（* 为当前分支）
git branch -a                   # 查看本地 + 远程所有分支
git branch <分支名>             # 创建分支
git checkout <分支名>           # 切换分支（旧版）
git switch <分支名>             # 切换分支（新版推荐）
git checkout -b <分支名>        # 创建并切换（旧版）
git switch -c <分支名>          # 创建并切换（新版推荐）

git merge <分支名>              # 将指定分支合并到当前分支
git merge --no-ff <分支名>      # 禁止快进合并，保留合并记录
git rebase <分支名>             # 变基：把当前分支的提交接到目标分支之后

git branch -d <分支名>          # 删除已合并的分支
git branch -D <分支名>          # 强制删除分支（未合并也删）
git branch -m <新名>            # 重命名当前分支
```

> 提示：`merge` 保留分支历史更真实；`rebase` 历史线性更干净，但**已推送到公共仓库的提交不要 rebase**。

---

## 五、远程协作

```bash
git remote -v                       # 查看远程仓库地址
git remote add origin <url>         # 添加远程仓库
git remote rename origin upstream   # 重命名远程
git remote remove <名称>            # 移除远程

git fetch origin                    # 拉取远程更新但不合并（安全）
git pull                            # 拉取并合并（等价 fetch + merge）
git pull --rebase                   # 拉取并以变基方式整合

git push origin <分支名>            # 推送指定分支
git push -u origin <分支名>         # 首次推送并建立跟踪关系，之后可直接 git push
git push --force-with-lease         # 需要覆盖远程时使用（比 --force 安全）
git push origin --delete <分支名>   # 删除远程分支

git branch -u origin/<分支名>       # 让本地分支跟踪远程分支
```

---

## 六、查看历史与搜索

```bash
git log                           # 查看提交历史
git log --oneline                 # 每条提交一行显示
git log --graph --oneline --all   # 图形化查看所有分支历史
git log -3                        # 只看最近 3 条
git log --author="<作者>"         # 按作者过滤
git log -p <文件>                 # 查看某文件的完整修改历史（含 diff）
git log --since="2 weeks ago"     # 按时间范围过滤
git log --stat                    # 附带每次提交的文件改动统计

git show <commit>                 # 查看某次提交的详细内容
git blame <文件>                  # 逐行显示最后修改人和提交
git reflog                        # 查看 HEAD 的所有变动记录（找回误删提交的救命命令）
git checkout <commit>             # 临时查看某个历史版本（detached HEAD）
git restore --source=<commit> -- <文件>  # 从指定提交恢复某个文件
```

---

## 七、标签（Tag）

```bash
git tag                          # 列出所有标签
git tag v1.0.0                   # 创建轻量标签
git tag -a v1.0.0 -m "发布说明"  # 创建附注标签（推荐）
git tag -a v1.0.0 <commit>       # 给历史提交补打标签
git show v1.0.0                  # 查看标签详情
git push origin v1.0.0           # 推送单个标签
git push origin --tags           # 推送所有标签
git tag -d v1.0.0                # 删除本地标签
git push origin :refs/tags/v1.0.0  # 删除远程标签
```

---

## 八、.gitignore 常用规则

在仓库根目录创建 `.gitignore` 文件，被匹配的文件不会被 Git 跟踪：

```gitignore
# 依赖与构建产物
node_modules/
dist/
build/
target/

# 日志与临时文件
*.log
*.tmp
.DS_Store

# 环境与敏感配置
.env
*.pem
```

> 注意：`.gitignore` 只对**未跟踪**的文件生效。若文件已被跟踪，需先 `git rm --cached <文件>` 再忽略。

---

## 九、高频场景速查

| 场景 | 操作 |
|---|---|
| 改乱了想回到上一个提交 | `git reset --hard HEAD~1`（会丢失修改，慎用） |
| 提交信息写错了 | `git commit --amend -m "正确的信息"`（未推送时） |
| 代码推错了想撤回远程 | `git revert <commit>` 后再 push（推荐），或 `git push --force-with-lease`（仅限个人分支） |
| 找"这行代码是谁改的" | `git blame <文件>` 或 `git log -p <文件>` |
| 误删了本地分支找不到提交 | `git reflog` 找到 commit id，再 `git branch <新分支名> <commit>` |
| 临时切分支但当前改动没写完 | `git stash` → 切分支干活 → 切回来 `git stash pop` |
| 拉代码提示有冲突 | 手动解决冲突文件后 `git add .`，再 `git commit`（merge 场景）或 `git rebase --continue`（rebase 场景） |
| 放弃 rebase | `git rebase --abort` |

---

## 十、建议与提醒

1. **提交要小要频繁**：一次提交只做一件事，说明写清楚"为什么改"。
2. **推送前先拉取**：公共分支上先 `git pull --rebase` 保持历史整洁。
3. **危险命令三思**：`reset --hard`、`clean -fd`、`push --force` 都可能丢数据，执行前确认分支和工作区状态。
4. **master/main 保护**：团队协作时建议通过分支 + 合并请求（PR/MR）进主干，避免直接 push。
5. **万一丢提交别慌**：`git reflog` 通常能找回最近 90 天内的所有提交痕迹。

---