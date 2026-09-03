# 作业三 · 数据可视化

## 任务目标

掌握 matplotlib 与 seaborn 的基本绘图方法，能用图表讲清楚数据里的结论。

## 任务要求

基于清洗后的数据，完成 **4 类图表**，每张图都要有标题、坐标轴标签：

1. **柱状图** — 展示分类变量的分布或排名
2. **折线图** — 展示随时间变化的趋势
3. **散点图** — 展示两个数值变量的相关性
4. **箱线图或热力图** — 展示分布特征或变量间相关矩阵

## 提交物

| 文件 | 说明 |
| --- | --- |
| `visualize.py` | 绘图脚本 |
| `output/*.png` | 导出的图片，分辨率不低于 150 DPI |
| `report.md` | 实验报告 |

## 报告需包含

- 每张图表对应的分析结论（不是描述图，而是回答"看出了什么"）
- 图表配色、字体选择的考量

## 中文乱码解决方案

Windows 环境常见坑，在绘图前加这两行：

```python
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei", "Microsoft YaHei"]  # 中文字体
plt.rcParams["axes.unicode_minus"] = False                        # 正常显示负号
```

macOS / Linux 可改用 `Arial Unicode MS` 或 `Noto Sans CJK SC`。

## 常用代码提示

```python
import seaborn as sns

sns.set_theme(style="whitegrid", font="SimHei")  # 统一样式

fig, ax = plt.subplots(figsize=(10, 6), dpi=150)
ax.bar(df["category"], df["value"])
ax.set_title("分类数值分布")
ax.set_xlabel("类别")
ax.set_ylabel("数值")
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig("output/bar_chart.png", dpi=150)
```
