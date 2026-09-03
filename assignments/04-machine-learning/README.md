# 作业四 · 机器学习建模

## 任务目标

掌握 scikit-learn 的标准建模流程，理解训练集/测试集划分与模型评估的意义。

## 任务要求

1. 自选一个**分类**或**回归**任务（可用 sklearn 内置数据集，如鸢尾花、波士顿房价、手写数字）
2. 完整走通以下流程：
   - 特征选择与预处理（标准化 / 独热编码）
   - 划分训练集与测试集（`train_test_split`，测试集占比 20%~30%）
   - 训练至少 **2 种**模型并对比效果
   - 使用合适的指标评估（分类看准确率、精确率、召回率、F1；回归看 MAE、RMSE、R²）
3. 输出模型对比表与结论

## 提交物

| 文件 | 说明 |
| --- | --- |
| `train.py` | 建模脚本 |
| `report.md` | 实验报告 |

## 报告需包含

- 任务定义与数据来源
- 特征工程说明
- 不同模型的评估指标对比表
- 结果分析与改进思路

## 常用代码提示

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, accuracy_score

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

scaler = StandardScaler().fit(X_train)      # 只在训练集上拟合，避免数据泄露
X_train_s = scaler.transform(X_train)
X_test_s = scaler.transform(X_test)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train_s, y_train)
y_pred = model.predict(X_test_s)

print(accuracy_score(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

## 易错提醒

- **数据泄露**：标准化必须先 `fit` 训练集再 `transform` 测试集，不能对全量数据统一 `fit`
- **类别不平衡**：分类任务记得用 `stratify=y` 保持类别比例
- **准确率陷阱**：类别不均衡时，准确率高不代表模型好，要看 F1
