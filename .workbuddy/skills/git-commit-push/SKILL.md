---
name: git-commit-push
description: 把本地仓库的改动走完 Git 全流程并推送到远端——状态检查、敏感文件体检、暂存、规范提交、同步远端、推送、核验，一次做完且全程无交互、不弹窗。当用户说“把改动提交并推送”“推一下仓库”“commit 然后 push”“同步到 GitHub”“帮我提交本地仓库”“推送这个目录”“提交并推送”等时使用。内置本机实测的代理降级顺序、免弹窗凭据参数、远端口径核验方式，以及遇到冲突或被拒绝时的停手规则。
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
| 凭据 | gh CLI 已登录（keyring，scope 含 `repo`）；`git credential fill` 可**静默**取到 token，正常推送不会弹窗 |
| credential.helper | GCM（`~/.gitconfig` 先置空再指向 PortableGit 的 git-credential-manager.exe） |
| 网络 | 直连与会话代理（`https_proxy=127.0.0.1:58506`）**均可能中途失效**；失效时代理报 `CONNECT tunnel failed 502`，直连报 `Failed to connect ... 443`。`127.0.0.1:10808` 不保证可用 |

**Bash 工具必须先修 PATH**，否则 `ls` / `head` / `timeout` / `dirname` 会 command not found：

```bash
export PATH="/usr/bin:/bin:/mingw64/bin:/cmd:$PATH"
```

## 二、三条铁律

**1. 绝不弹窗。** 任何可能触发认证的命令，一律加无交互前缀：

```bash
GIT_TERMINAL_PROMPT=0 GCM_INTERACTIVE=Never git -c credential.interactive=never <cmd>
```

凭据已缓存，正常根本走不到交互分支；加前缀是防止凭据失效时挂起或弹 GUI。

**2. 网络先重试，再降级，且必须设超时。** 实测网络会**抖动**：同一分钟内 `push` 报 `CONNECT tunnel failed, response 502`，紧接着 `ls-remote` 又成功。所以先原地重试，再走降级链：

1. **原样重试 2–3 次**（抖动居多，往往第二次就过）；
2. 仍失败 → **清代理重试**（会话代理最常见的故障是 502）：
   `env -u https_proxy -u http_proxy -u HTTPS_PROXY -u HTTP_PROXY git -c http.proxy= -c https.proxy= <cmd>`
3. 仍失败 → 试 `https_proxy=http://127.0.0.1:10808`（不保证可用）；
4. 全失败 → **停手**报告原始报错，别继续瞎试。

每一步都用 `timeout` 兜底，避免长时间挂起（实测直连超时能卡 21 秒）：

```bash
timeout 30 git <cmd>
```

**3. 推送结果以远端为准。** 沙箱内 `refs/remotes/origin/*` **写不进去**（见 §四），本地 `origin/main` 不可信。判断是否推成功，只认 `git ls-remote`。

## 三、标准流程

### S0 前置检查

```bash
git rev-parse --show-toplevel        # 确认在仓库内
git status -sb                       # 当前分支 + 改动清单
git remote -v                        # 确认 origin
git config user.name; git config user.email
```

- 分支名以 `git status -sb` 输出为准，**不要假定是 main**。
- 工作区无改动时，仍要用 `git ls-remote` 比对本地是否已推（本地 ahead/behind 不可信）。已同步就直接报告「无改动，无需推送」并结束。

### S1 审视改动 + 敏感文件体检

```bash
git status -s
git diff                             # 未暂存的改动
git diff --cached                    # 已暂存的改动（若有）
```

**推送前必须扫一遍改动清单**，出现下列内容先停手报告，不要自作主张提交：

- **密钥凭据**：`.env`、`*.key`、`*.pem`、`id_rsa`、`token`、含 `password=` 的文件；
- **大文件**：单个 >5MB 要点出来（GitHub 单文件硬上限 100MB）。本用户既有习惯是 >5MB 素材放坚果云，不进仓库；
- **数据与产物**：`data/` 下的大数据集、`__pycache__/`、`.venv/`、`*.ipynb_checkpoints` —— 应进 `.gitignore`。

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
git commit -m "chore(skill): 新增本地仓库提交推送 skill"
```

- 禁止 `update`、`修改`、`asdf` 这类无追溯价值的信息；
- 不写 emoji，不加 AI 署名 / `Co-Authored-By`；
- 中文信息若在 Bash 工具里出现乱码，改用 UTF-8 文件：先写临时文件再 `git commit -F <tmpfile>`。

### S4 同步远端（先 pull 后 push）

```bash
git fetch origin                     # 加 timeout 与降级前缀
git ls-remote origin refs/heads/main # 远端当前值
```

- 远端值 ≠ 本地 HEAD 且本地 HEAD 是其祖先 → 落后，需同步：
  `git pull --rebase origin main`
- 已一致 → 跳过，不要无脑 rebase。
- 若 rebase 上游解析失败（`origin/main` 读不到），改用显式目标：
  `git rebase FETCH_HEAD`（fetch 后 FETCH_HEAD 始终可用）。

### S5 推送

```bash
timeout 60 git push origin main
```

未跟踪上游的新分支首次推送用 `git push -u origin <branch>`。

### S6 核验（关键：问远端，不看本地）

```bash
echo "local  = $(git rev-parse HEAD)"
echo "remote = $(timeout 30 git ls-remote origin refs/heads/main | cut -f1)"
timeout 30 git ls-remote origin refs/heads/main | cut -f1 | grep -q "^$(git rev-parse HEAD)$" \
  && echo "✅ 推送成功" || echo "❌ 未推上去"
```

两个哈希一致才算完成。**不要用 `git status -sb` 的 ahead/behind 或 `git rev-parse origin/main` 判断**——本环境下它们是陈旧/缺失的（§四），会导致把成功的推送误判为失败。

## 四、已知环境现象（别当故障，别去修）

在 WorkBuddy 沙箱内实测确认：

- **`refs/remotes/origin/*` 无法建立。** `git update-ref refs/remotes/origin/main <sha>` 返回 exit=0，但目录与文件都不会生成；`git fetch` 也照样报 `* [new branch] main -> origin/main` 却什么都没写。**已在全新临时仓库复现，属环境级限制，不是本仓库损坏。**
- 后果：`git status -sb` 可能显示假的 `[ahead N]` 或 `[gone]`；`origin/main` 缺失或陈旧。
- **应对**：一律用 `git ls-remote` 判定远端状态（§二.3、§六）；不要为此反复 fetch、重建仓库或改 `.git`。
- **可选修复**（让 `git status` 好看一点，仅在本机原生终端做）：把正确的哈希写进 `.git/packed-refs` 的 `refs/remotes/origin/main` 行。这是显示层修补，不影响推送本身；远端前进后该值会陈旧，属正常。
- 在用户自己的 Git Bash / 终端里没有这个限制。

## 五、故障处理

| 现象 | 处理 |
| --- | --- |
| `CONNECT tunnel failed, response 502` | 先重试 2–3 次（多为抖动）；仍不行 → 降级②清代理重试 |
| `Failed to connect to github.com:443` | 直连不通 → 降级③试 10808；仍失败则停手报告 |
| `! [rejected] ... (fetch first)` / non-fast-forward | `git pull --rebase origin main` 后重推；**不要** force |
| `CONFLICT` | **停手**。列出冲突文件，让用户决定保留哪边，不要自动选边 |
| 提示输入用户名/密码 | 凭据失效。**停手报告**，不要尝试在会话里输入密码，提示用户执行 `gh auth login` |
| `origin/main` 读不到 / `[gone]` | 正常现象，见 §四。改用 `git ls-remote` 判定 |
| 文件 >100MB 被拒 | 停手报告。选项：移出仓库（坚果云）或改用 Git LFS |
| 误提交了不该提交的文件（尚未 push） | `git reset --soft HEAD~1` → 补 `.gitignore` → 重新提交 |

## 六、禁止未经确认的操作

以下命令**必须先取得用户同意**，不得自行执行：

`git push --force` / `--force-with-lease`、`git reset --hard`、`git clean -fd`、`git branch -D`、`git filter-repo`、任何改写已推送历史的操作。

已推送的提交要撤销，只用 `git revert`。

## 七、输出约定

完成后回**一行结论**即可，不复述命令：

```
已推送 <短哈希> <提交信息> → origin/main（N 个文件，+A/-D）
```

被拦下时，说清三件事：卡在哪一步、原始报错、需要用户做的选择。
