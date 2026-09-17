---
name: git-commit-push
description: 把本地仓库的改动走完 Git 全流程并推送到远端——状态检查、敏感文件体检、暂存、规范提交、同步远端、推送、核验，一次做完且全程无交互、不弹窗。当用户说“把改动提交并推送”“推一下仓库”“commit 然后 push”“同步到 GitHub”“帮我提交本地仓库”“推送这个目录”“提交并推送”等时使用。内置本机实测的代理降级顺序、免弹窗凭据参数，以及遇到冲突或被拒绝时的停手规则。
agent_created: true
---

# Git Commit & Push（本地仓库提交推送）

把一个本地 git 仓库的改动安全地走完 `status → add → commit → sync → push → verify`，端到端一次做完，全程不弹窗、可复现、可核验。

## 一、环境事实（本机实测，别凭记忆改）

| 项 | 实测值（2026-09-17） |
| --- | --- |
| git | 2.55.0.windows.3（WorkBuddy PortableGit） |
| 本仓库远端 | `https://github.com/yangjh-xbmu/bigdata-ai-assignments.git`，origin，默认分支 `main` |
| 全局身份 | `yangjh-xbmu` / `yangjh@xbmu.edu.cn` |
| 凭据 | gh CLI 已登录（keyring，scope 含 `repo`）；`git credential fill` 可**静默**取到 token |
| credential.helper | GCM（`~/.gitconfig` 先置空再指向 PortableGit 的 git-credential-manager.exe） |
| 网络 | **直连 GitHub 可用**；会话注入的 `https_proxy=127.0.0.1:58506` 也可用；`127.0.0.1:10808` 当前**不通** |

**Bash 工具必须先修 PATH**，否则 `ls` / `head` / `dirname` 会 command not found：

```bash
export PATH="/usr/bin:/bin:/mingw64/bin:/cmd:$PATH"
```

## 二、两条铁律

**1. 绝不弹窗。** 任何可能触发认证的命令，一律加无交互前缀：

```bash
GIT_TERMINAL_PROMPT=0 GCM_INTERACTIVE=Never git -c credential.interactive=never <cmd>
```

凭据已缓存，正常情况根本走不到交互分支；加前缀是防止凭据失效时挂起或弹 GUI。

**2. 代理不硬编码。** 按此顺序降级，命中即停：

1. 直接用当前环境跑；
2. 失败（连不上／超时）→ 清代理重试：`git -c http.proxy= -c https.proxy= ...`，必要时再 `env -u https_proxy -u http_proxy -u HTTPS_PROXY -u HTTP_PROXY`；
3. 仍失败才试 `https_proxy=http://127.0.0.1:10808`（不保证可用）；
4. 三次都失败就**停下**报告原始报错，不要继续瞎试。

## 三、标准流程

### S0 前置检查

```bash
git rev-parse --show-toplevel        # 确认在仓库内
git status -sb                       # 当前分支 + 领先/落后 + 改动清单
git remote -v                        # 确认 origin
git config user.name; git config user.email
```

- 分支名以 `git status -sb` 输出为准，**不要假定是 main**。
- 若已是 `## main...origin/main` 且工作区无改动 → 直接报告「无改动，无需推送」并结束，不要空跑一轮。

### S1 审视改动 + 敏感文件体检

```bash
git status -s
git diff                             # 未暂存的改动
git diff --cached                    # 已暂存的改动（若有）
```

**推送前必须扫一遍改动清单**，出现下列内容先停手报告，不要自作主张提交：

- **密钥凭据**：`.env`、`*.key`、`*.pem`、`id_rsa`、`token`、含 `password=` 的文件；
- **大文件**：单个 >5MB 要点出来（GitHub 单文件硬上限 100MB）。本用户的既有习惯是 >5MB 的素材放坚果云，不进仓库；
- **数据与产物**：`data/` 下的大数据集、`__pycache__/`、`.venv/`、`*.ipynb_checkpoints` —— 这类应进 `.gitignore`。

### S2 暂存

优先**指定路径**，不要无条件 `git add .`：

```bash
git add -- <路径1> <路径2>
```

只有用户明确说「全部提交」且 S1 体检干净时，才用 `git add -A`。

### S3 提交

格式遵循本仓库 `docs/submission-guide.md`：`类型(范围): 中文描述`

| 类型 | 用途 |
| --- | --- |
| feat | 新增内容／功能 |
| fix | 修复错误 |
| docs | 只改文档 |
| refactor | 重构，不改变行为 |
| test | 新增或修改测试 |
| chore | 构建、依赖、配置 |

```bash
git commit -m "docs(skill): 新增本地仓库提交推送 skill"
```

- 禁止 `update`、`修改`、`asdf` 这类无追溯价值的信息；
- 不写 emoji，不加 AI 署名 / `Co-Authored-By`；
- 中文信息若在 Bash 工具里出现乱码，改用 UTF-8 文件：先写临时文件再 `git commit -F <tmpfile>`。

### S4 同步远端（先 pull 后 push）

```bash
git fetch origin
git status -sb                       # 看是否 behind
# 仅在落后时执行：
git pull --rebase origin main
```

没落后就跳过 rebase，不要无脑执行。

### S5 推送

```bash
git push origin main
```

未跟踪上游的新分支首次推送用 `git push -u origin <branch>`。

### S6 核验

```bash
git status -sb                 # 期望 ## main...origin/main，且无 ahead/behind
git log --oneline -1
git rev-parse HEAD
git rev-parse origin/main      # 应与 HEAD 一致
```

两者一致才算完成。别只看 push 命令的退出码。

## 四、故障处理

| 现象 | 处理 |
| --- | --- |
| `! [rejected] ... (fetch first)` / non-fast-forward | `git pull --rebase origin main` 后重推；**不要** force |
| `CONFLICT` | **停手**。列出冲突文件，让用户决定保留哪边，不要自动选边 |
| 连不上 github.com / 超时 | 按 §二.2 降级顺序重试 |
| 提示输入用户名/密码 | 凭据失效。**停手报告**，不要尝试在会话里输入密码，提示用户执行 `gh auth login` |
| 文件 >100MB 被拒 | 停手报告。选项：移出仓库（坚果云）或改用 Git LFS |
| 误提交了不该提交的文件（尚未 push） | `git reset --soft HEAD~1` → 补 `.gitignore` → 重新提交 |

## 五、禁止未经确认的操作

以下命令**必须先取得用户同意**，不得自行执行：

`git push --force` / `--force-with-lease`、`git reset --hard`、`git clean -fd`、`git branch -D`、`git filter-repo`、任何改写已推送历史的操作。

已推送的提交要撤销，只用 `git revert`。

## 六、输出约定

完成后回**一行结论**即可，不复述命令：

```
已推送 <短哈希> <提交信息> → origin/main（N 个文件，+A/-D）
```

被拦下时，说清三件事：卡在哪一步、原始报错、需要用户做的选择。
