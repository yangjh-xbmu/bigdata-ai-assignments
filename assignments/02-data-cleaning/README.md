# 作业二 · 数据清洗

## 任务目标

掌握使用 pandas 处理"脏数据"的完整流程，理解数据质量对后续分析的影响。

## 任务要求

1. 基于作业一采集的数据，或自带一份含缺陷的数据集
2. 依次完成以下处理，每一步都要有代码与说明：
   - **重复值检测与删除**
   - **缺失值处理**（删除 / 均值填充 / 中位数填充，说明选择理由）
   - **数据类型转换**（如字符串日期转 `datetime`、文本数值转 `float`）
   - **异常值检测**（箱线图或 Z-score 方法）
   - **格式规范化**（去空格、统一单位、统一大小写）
3. 输出清洗前后的数据对比统计

## 提交物

| 文件 | 说明 |
| --- | --- |
| `clean.py` | 清洗脚本 |
| `data/cleaned_data.csv` | 清洗后的数据 |
| `report.md` | 实验报告 |

## 报告需包含

- 清洗前后记录数、缺失率的变化对比表
- 每种缺失值处理方式的取舍理由
- 异常值的判定标准与处理方式

## 常用代码提示

```python
import pandas as pd

df = pd.read_csv("data/raw_data.csv")
print(df.info())              # 查看字段类型与非空计数
print(df.isna().sum())        # 统计各列缺失值
print(df.describe())          # 数值列分布概况

df = df.drop_duplicates()     # 删除完全重复的行
df["date"] = pd.to_datetime(df["date"], errors="coerce")  # 日期转换
df["value"] = df["value"].fillna(df["value"].median())    # 中位数填充
```
