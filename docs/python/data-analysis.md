---
title: Python 数据分析
tags:
  - Python
  - 数据分析
  - pandas
  - matplotlib
---

## 介绍 / Introduction

使用 Python 进行数据清洗、统计分析与可视化，核心栈为 pandas + numpy + matplotlib。

!!! note
    推荐在 Jupyter Lab 中交互式运行，便于观察中间结果。

## 基础流程

```bash
pip install pandas numpy matplotlib seaborn jupyterlab
```

```python
import pandas as pd
import matplotlib.pyplot as plt

# 读取 CSV
df = pd.read_csv("data.csv")

# 查看基本信息
print(df.head())
print(df.describe())

# 分组聚合
summary = df.groupby("category")["amount"].sum()
summary.plot(kind="bar")
plt.title("Amount by Category")
plt.tight_layout()
plt.show()
```
