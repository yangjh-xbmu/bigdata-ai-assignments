# 作业五 · 深度学习入门

## 任务目标

理解神经网络的基本原理，能搭建并训练一个简单的神经网络模型。

## 任务要求

1. 完成一个经典入门任务（二选一）：
   - **图像**：MNIST 手写数字识别
   - **文本**：新闻或影评的情感二分类
2. 搭建一个至少包含 **2 个隐藏层**的全连接网络
3. 记录训练过程中的损失与准确率变化，绘制训练曲线
4. 与作业四的传统机器学习模型做效果对比

## 提交物

| 文件 | 说明 |
| --- | --- |
| `train.py` | 训练脚本 |
| `output/training_curve.png` | 训练曲线图 |
| `report.md` | 实验报告 |

## 报告需包含

- 网络结构设计（各层维度、激活函数、参数量）
- 超参数选择（学习率、批大小、训练轮数）及调整过程
- 与传统机器学习模型的对比结论
- 是否出现过拟合、如何应对

## 环境说明

PyTorch 体积较大，按需安装（CPU 版本即可）：

```bash
pip install torch --index-url https://download.pytorch.org/whl/cpu
```

若安装困难，可改用 `sklearn.neural_network.MLPClassifier` 完成本作业，流程一致。

## 常用代码提示

```python
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(28 * 28, 256),
            nn.ReLU(),
            nn.Linear(256, 128),
            nn.ReLU(),
            nn.Linear(128, 10),
        )

    def forward(self, x):
        return self.net(x)
```

## 易错提醒

- 输入维度要与实际数据对齐（MNIST 是 1×28×28，展平后 784）
- 分类任务最后一层**不加激活函数**，直接输出 logits 交给 `CrossEntropyLoss`
- 训练前确认是否已切换到 GPU：`device = torch.device("cuda" if torch.cuda.is_available() else "cpu")`
