# 大数据与人工智能 · 课程作业仓库

> Course assignments for **Big Data & Artificial Intelligence**
> 仓库地址：<https://github.com/yangjh-xbmu/bigdata-ai-assignments>

本仓库用于存放《大数据与人工智能》课程的全部作业代码、数据与实验报告。

## 目录结构

```
bigdata-ai-assignments/
├── assignments/          # 作业主目录，每次作业一个子目录
│   ├── 01-data-collection/    # 作业一：数据采集
│   ├── 02-data-cleaning/      # 作业二：数据清洗
│   ├── 03-visualization/      # 作业三：数据可视化
│   ├── 04-machine-learning/   # 作业四：机器学习建模
│   └── 05-deep-learning/      # 作业五：深度学习入门
├── notebooks/            # Jupyter 实验性代码、课堂演示
├── data/                 # 数据集（大文件不入库，见下方说明）
├── docs/                 # 课程文档与提交规范
└── requirements.txt      # Python 依赖清单
```

## 环境准备

```bash
# 1. 克隆仓库（首次）
git clone https://github.com/yangjh-xbmu/bigdata-ai-assignments.git
cd bigdata-ai-assignments

# 2. 创建虚拟环境
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux

# 3. 安装依赖
pip install -r requirements.txt
```

## 作业提交流程

每次写作业，标准动作用四条命令：

```bash
git pull                                   # 1. 先拉取，避免冲突
git add .                                  # 2. 暂存改动
git commit -m "feat(01): 完成数据采集作业"  # 3. 提交
git push                                   # 4. 推送到 GitHub
```

### 提交信息规范

格式：`类型(作业编号): 简短描述`

| 类型 | 用途 |
| --- | --- |
| `feat` | 新增作业内容或功能 |
| `fix` | 修复错误 |
| `docs` | 只改文档 |
| `refactor` | 重构代码，不改变功能 |
| `test` | 新增或修改测试 |

示例：`fix(03): 修正柱状图中文乱码问题`

### 文件命名规范

- 作业目录：`两位数字-英文名`，如 `01-data-collection`
- 脚本文件：小写英文 + 下划线，如 `collect_weather.py`
- 报告文件：`report.md` 或 `report.pdf`，放在对应作业目录内

## 数据文件处理原则

- **大于 5 MB 的数据集不要提交到 Git**，已在 `.gitignore` 中屏蔽常见大文件格式
- 数据集统一放在 `data/` 目录，并在作业 README 中注明数据来源与获取方式
- 若必须共享大数据，请使用网盘链接并在文档中标注

## 注意事项

- 每次动手写代码前先执行 `git pull`，这是避免冲突最有效的一步
- 不要提交虚拟环境目录（`.venv/`）、密钥、或个人配置文件
- 作业中的敏感信息（学号、手机号）请先脱敏再提交

## 许可

本项目采用 [MIT License](./LICENSE)。
