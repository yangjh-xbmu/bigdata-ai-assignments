# 推送到 GitHub 指南（自建公开仓库）

本地仓库已初始化并提交（分支 `main`）。下面任选一种方式，把仓库推送为**公开（Public）**仓库。
建议仓库名：`concept-learning-materials`（与 README 中引用一致）。

---

## 方式 A：使用 GitHub CLI（最省事，若本机已安装 `gh`）

```bash
cd C:/Users/houti/WorkBuddy/2026-09-06-20-46-13
gh repo create concept-learning-materials --public --source=. --remote=origin --push
```

该命令会自动：在 GitHub 创建公开仓库 → 添加 `origin` 远程 → 推送 `main` 分支。

---

## 方式 B：手动创建（HTTPS，用 PAT 作为密码）

1. 在 GitHub 网页新建 **Public** 仓库，名称 `concept-learning-materials`，**不要**勾选 "Initialize with README"。
2. 在终端执行：

```bash
cd C:/Users/houti/WorkBuddy/2026-09-06-20-46-13
git remote add origin https://github.com/houtiweiwei/concept-learning-materials.git
git branch -M main
git push -u origin main
```

> 推送时若提示输入密码，请使用 **Personal Access Token（PAT，需 `repo` 权限）** 而非 GitHub 登录密码（GitHub 已禁用密码方式 git 认证）。

---

## 方式 C：SSH（若你本机 SSH 密钥已绑定 GitHub）

```bash
cd C:/Users/houti/WorkBuddy/2026-09-06-20-46-13
git remote add origin git@github.com:houtiweiwei/concept-learning-materials.git
git push -u origin main
```

> 注意：本作业使用的 WorkBuddy 沙盒环境屏蔽了 SSH 22/443 端口，因此沙盒内无法走 SSH；请在你的本机终端执行以上命令。

---

## 推送后自查清单

- [ ] 仓库为 **Public**（教师无需申请权限即可访问）。
- [ ] 能看到 `.workbuddy/skills/concept-learning-generator/SKILL.md`。
- [ ] 能看到 `learning-materials/` 下的三份 HTML 与 `concept-relationship.md`、`README.md`、`.gitignore`。
- [ ] 仓库**不含**任何密钥/隐私文件（已被 `.gitignore` 排除）。
- [ ] 提交记录中存在本地的 `Initial commit`。
