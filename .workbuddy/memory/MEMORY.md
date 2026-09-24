# bigdata-ai-assignments 项目约定（宏哥）

## 学习类 HTML 资料的制作原则（2026-09-03 明确）
- **图形优先、文字极少**：讲解用自绘 SVG 示意图表达，能用图就不写字；每节一句话标题 + 一句图注即可。
- **讲清楚概念是唯一目标**：不堆砌权威来源、不展示"我查了多少资料"（文末可留一行轻量出处，不作主体）。
- **严肃不幼稚**：面向严肃学习者，可以用"菜谱/入职手册/抽屉"等生活化类比与几何示意图，但**禁用卡通/漫画/可爱脸谱**。
- **设计克制**：中性灰 + 单一强调色，无图标字体、无装饰性配色；浅色底。
- **测试分级**：概念 → 辨析 → 应用场景 三级递进，覆盖概念记忆与能力运用；每题带解析，即时判分，查看答案不计分。
- 样例成品：`docs/agent-skills-learning-guide.html`（Agent Skill 图解学习资料，v2 图文化改版）。
- **测验交互**：测试题不要一次全部展示，必须一道一道线性出现（上一题/下一题），即时判分；「看答案」不计分；全部完成后再出分级与总分成绩。
- **交付前自查**：生成 HTML 后先自己截图检查排版（文字溢出、图形重叠、错位）再交付；发现布局错误需修复。

## 项目级 Skill
- 本仓库 `.workbuddy/skills/visual-lesson-builder/`：图解学习资料生成器（上述全部约定 + 一题一题测验引擎模板 assets/starter-template.html）。在**当前仓库内**直接复用。
  - ⚠️ 2026-09-24 发现：该目录在磁盘上已不存在，git 里显示为两个文件「已删除且未提交」。此前记录与现状不符，需要时从 git 恢复或重建。
- 本仓库 `.workbuddy/skills/git-commit-push/`：本地仓库提交推送（status → add → commit → fetch → push → 核验），含免弹窗参数、代理降级顺序、冲突停手规则。触发语："提交推送""推一下仓库""commit 然后 push"。

## Python 基础资料现状（docs/python-basics/）
- 已完成：`01-variables-types-io.html`、`01b-operators.html`（运算符专题，2026-09-24 新增）、`02-strings-lists.html`、`03-conditionals-loops.html`，以及目录页 `index.html`（2026-09-24 补建，此前是长期死链）。
- 待制作：04 字典·集合·推导式 / 05 函数 / 06 模块标准库pip / 07 文件读写异常 / 08 类与对象调试。
- 编号约定：新增专题用字母后缀（`01b`）插入，不整体重编号——避免改动后续文件名与互链。
- 测验选项样式陷阱：`.ltr` 必须 `display:inline-block;width:18px`，否则紧跟 `<code>` 时字母与内容贴合（渲染成 `A3`）。01–03 已于 2026-09-24 统一修复。

## 截图自查方法（生成 HTML 后用）
- Chrome 路径：`~/.agent-browser/browsers/chrome-153.0.8010.36/chrome.exe`。
- 命令：`--headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=2 --window-size=880,1150 --screenshot=...`。
- **坑**：每次用新的 `--user-data-dir` 会挂起不出图，必须复用同一个 profile 目录。
- 逐屏核对：临时复制一份 HTML 注入 `<style>html{margin-top:-Npx}</style>` 再截，截完删除。别留在仓库里。

## Git 环境事实（2026-09-17 实测，skill 依赖这些结论）
- 远端 `https://github.com/yangjh-xbmu/bigdata-ai-assignments.git`，分支 main；身份 yangjh-xbmu / yangjh@xbmu.edu.cn。
- WorkBuddy 沙箱内 **`refs/remotes/origin/*` 写不进去**：`update-ref`/`fetch` 都报成功但不落地（已用全新临时仓库复现，属环境级）。因此 `git status -sb` 的 ahead/behind、`origin/main` 都不可信，**判定推送结果必须用 `git ls-remote origin refs/heads/main`**。用户自己的终端没这个问题。
- `.git/packed-refs` 里的 `refs/remotes/origin/main` 是可读源；要修 `git status` 显示，直接改写这一行即可（显示层修补，不影响推送）。
- Bash 工具里 PATH 缺 coreutils，任何命令前先 `export PATH="/usr/bin:/bin:/mingw64/bin:/cmd:$PATH"`。
- 提交信息规范见 `docs/submission-guide.md`：`类型(范围): 中文描述`，禁 emoji、禁 AI 署名。
