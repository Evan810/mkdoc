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

## 安装依赖

```bash
pip install pandas numpy matplotlib seaborn jupyterlab
```

## numpy 基础

```python
import numpy as np

a = np.array([1, 2, 3, 4])
a.shape          # (4,)
a.dtype          # int64
a.reshape(2, 2)  # 变形
a.mean(), a.sum(), a.std()   # 均值 / 求和 / 标准差

# 广播与向量化
b = np.array([[1], [2]])
(a + 1) * 2      # 全元素运算，无需循环

# 常用生成
np.zeros((2, 3))         # 全 0
np.ones((2, 3))          # 全 1
np.arange(0, 10, 2)      # 等差：[0 2 4 6 8]
np.linspace(0, 1, 5)     # 等分
np.random.rand(3)        # 0~1 随机
np.random.randn(3)       # 标准正态
```

## 数据读取

```python
import pandas as pd

df = pd.read_csv("data.csv")                                  # CSV
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")          # Excel
df = pd.read_json("data.json")                                # JSON
df = pd.read_sql("SELECT * FROM t", conn)                     # 数据库

df.to_csv("out.csv", index=False, encoding="utf-8-sig")       # 导出（防 Excel 乱码）
```

## DataFrame 速览

```python
df.head()             # 前 5 行
df.tail(3)            # 后 3 行
df.info()             # 结构与类型
df.describe()         # 数值统计
df.shape              # (行, 列)
df.columns            # 列名
df.dtypes             # 各列类型

df["amount"]          # 取一列（Series）
df[["name", "amount"]]        # 取多列
df.loc[0, "name"]             # 按标签定位
df.iloc[0, 1]                 # 按位置定位
df[df["amount"] > 100]        # 条件筛选
df.query("amount > 100 and city == '武汉'")   # 表达式筛选

# 新增 / 修改列
df["tax"] = df["amount"] * 0.1
df.rename(columns={"amount": "总额"}, inplace=True)
df.drop(columns=["tax"], inplace=True)
```

## 清洗与缺失值

```python
df.isnull().sum()                  # 各列缺失数量
df.dropna()                        # 删除含缺失的行
df.fillna(0)                       # 填充固定值
df["age"].fillna(df["age"].mean()) # 填充均值

df.duplicated().sum()              # 重复行数量
df.drop_duplicates(inplace=True)   # 去重

df["amount"].astype(float)         # 类型转换
df["date"] = pd.to_datetime(df["date"])   # 转日期
```

## 分组与聚合

```python
# 单列聚合
df.groupby("category")["amount"].sum()
df.groupby("category")["amount"].agg(["sum", "mean", "count"])

# 多列分组
df.groupby(["city", "category"])["amount"].mean()

# 透视表
pd.pivot_table(df, values="amount", index="city",
               columns="category", aggfunc="sum", fill_value=0)
```

## 合并与拼接

```python
pd.concat([df1, df2], ignore_index=True)          # 纵向拼接
df1.merge(df2, on="id", how="left")               # 关联（left/right/inner/outer）
df.join(other, on="id")                           # 按索引连接
```

## 排序与统计

```python
df.sort_values("amount", ascending=False)         # 排序
df["amount"].sum(), df["amount"].mean()           # 求和 / 均值
df["amount"].median(), df["amount"].std()         # 中位数 / 标准差
df["category"].value_counts()                     # 计数
df.corr(numeric_only=True)                        # 相关系数矩阵
```

## 可视化

```python
import matplotlib.pyplot as plt
import seaborn as sns

plt.rcParams["font.sans-serif"] = ["SimHei"]      # 中文显示
plt.rcParams["axes.unicode_minus"] = False        # 负号显示

# 折线图
df.plot(x="date", y="amount", kind="line")
plt.title("趋势")
plt.tight_layout()
plt.show()

# 柱状图（分组聚合）
summary = df.groupby("category")["amount"].sum()
summary.plot(kind="bar")

# 常用图形
df["amount"].plot(kind="hist", bins=20)           # 直方图
df.plot(kind="scatter", x="price", y="sales")     # 散点图

# seaborn
sns.barplot(data=df, x="category", y="amount")
sns.heatmap(df.corr(numeric_only=True), annot=True)
sns.pairplot(df)
```

## 进阶技巧

```python
# apply 自定义函数
df["level"] = df["amount"].apply(lambda x: "高" if x > 1000 else "低")

# 分箱
pd.cut(df["amount"], bins=[0, 100, 500, 10000],
       labels=["低", "中", "高"])

# 时间序列
df.set_index("date").resample("M")["amount"].sum()   # 按月汇总
df["month"] = df["date"].dt.month                    # 提取月份
```

!!! tip
    分析流程建议：读取 → 概览（info/describe）→ 清洗（缺失/重复/类型）→ 转换（派生列）→ 聚合/透视 → 可视化 → 导出结论。
