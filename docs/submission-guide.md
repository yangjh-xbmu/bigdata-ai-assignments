# Git 操作速查 · 作业提交指南

面向本仓库的日常使用，涵盖克隆、编辑、提交、推送的完整流程与常见故障处理。

---

## 一、首次使用（只做一次）

```bash
# 1. 克隆仓库
git clone https://github.com/yangjh-xbmu/bigdata-ai-assignments.git
cd bigdata-ai-assignments

# 2. 配置身份（若本机未配置过）
git config user.name  "你的名字"
git config user.email "你的邮箱"
```

> Windows 上首次推送会弹出 GitHub 登录窗口，用浏览器完成授权即可，
> 之后凭据由 Git Credential Manager 自动保存，无需重复登录。

---

## 二、每次写作业的固定四步

```bash
git pull                                     # ① 拉取最新代码
# ... 编写或修改作业文件 ...
git add .                                    # ② 暂存所有改动
git commit -m "feat(01): 完成数据采集作业"   # ③ 提交到本地
git push                                     # ④ 推送到 GitHub
```

**为什么第一步必须是 `git pull`**：多人协作或换电脑操作时，远端可能已有新提交。
先拉取能让你在本地解决冲突，而不是在推送时被拒绝。

---

## 三、随时查看状态

```bash
git status              # 查看哪些文件被改动、哪些待提交
git status -s           # 精简版输出
git diff                # 查看尚未暂存的具体改动内容
git log --oneline -10   # 查看最近 10 条提交记录
```

### 文件状态速查

| 状态 | 含义 |
| --- | --- |
| 红色 `??` | 未跟踪的新文件 |
| 红色 `M` | 已跟踪文件的修改，尚未暂存 |
| 绿色 `M` | 已暂存，等待提交 |

---

## 四、撤销操作

```bash
git restore <文件>              # 丢弃工作区的修改（未 add 时使用）
git restore --staged <文件>     # 把文件从暂存区撤回（已 add 但未 commit）
git reset --soft HEAD~1         # 撤销上一次 commit，改动保留在暂存区
git reset --hard HEAD~1         # 撤销上一次 commit，改动全部丢弃（慎用）
```

> `git reset --hard` 会永久删除未提交的改动，执行前先确认 `git status` 是干净的。
> 已经 `push` 到远端的提交，不要用 `reset` 撤销，改用 `git revert <commit-id>`。

---

## 五、常见问题与处理

### 1. `push` 被拒绝（non-fast-forward）

```
! [rejected] main -> main (fetch first)
```

原因：远端有你本地没有的新提交。处理：

```bash
git pull --rebase    # 把你的提交"接"在远端最新提交之后
git push
```

### 2. 产生合并冲突

`git pull` 后提示 `CONFLICT`，打开冲突文件会看到：

```text
<<<<<<< HEAD
你的改动
=======
远端他人的改动
>>>>>>> abc1234
```

处理：手动编辑文件，保留正确内容并**删掉这三行标记**，然后：

```bash
git add <冲突文件>
git commit -m "fix: 解决合并冲突"
git push
```

### 3. 误提交了大文件

若文件尚未 push：

```bash
git reset --soft HEAD~1        # 撤销提交
# 把大文件加入 .gitignore
git add .
git commit -m "chore: 移除大文件"
```

若已 push，需要清理历史，建议先用 `git filter-repo` 或联系仓库管理员处理。

### 4. 提交了不该提交的文件（如 `.venv/`）

```bash
echo ".venv/" >> .gitignore
git rm -r --cached .venv       # 从版本控制中移除，但保留本地文件
git commit -m "chore: 忽略虚拟环境目录"
git push
```

### 5. 想临时保存未完成的改动

```bash
git stash          # 把当前改动暂存起来，工作区恢复干净
git stash pop      # 之后恢复这些改动
```

---

## 六、提交信息规范

格式：`类型(作业编号): 简短描述`

| 类型 | 用途 |
| --- | --- |
| `feat` | 新增作业内容或功能 |
| `fix` | 修复错误 |
| `docs` | 只改文档 |
| `refactor` | 重构代码，不改变功能 |
| `test` | 新增或修改测试 |
| `chore` | 构建、依赖、配置类改动 |

正确示例：

```text
feat(02): 完成缺失值处理与异常值检测
fix(03): 修正柱状图中文乱码
docs: 补充作业四的评估指标说明
```

避免：`update`、`修改`、`asdf` 这类无法追溯含义的信息。

---

## 七、命令速查表

| 命令 | 作用 |
| --- | --- |
| `git clone <url>` | 克隆远程仓库 |
| `git status` | 查看工作区状态 |
| `git add <文件>` / `git add .` | 暂存指定文件 / 全部改动 |
| `git commit -m "信息"` | 提交到本地仓库 |
| `git push` | 推送到远程仓库 |
| `git pull` | 拉取并合并远端更新 |
| `git log --oneline` | 查看提交历史 |
| `git diff` | 查看未暂存的改动 |
| `git branch` | 查看分支 |
| `git checkout -b <分支名>` | 新建并切换到分支 |
| `git stash` / `git stash pop` | 临时保存 / 恢复改动 |

---

## 八、一条原则

**先 pull，后 push；小步提交，写清信息。**

提交粒度越小、说明越清楚，出问题时越容易定位和回退。
