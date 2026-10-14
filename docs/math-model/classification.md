---
title: 数学建模 - 分类模型
tags:
  - 数学建模
  - 分类
  - 机器学习
---

## 介绍 / Introduction

分类模型涵盖逻辑回归、决策树、SVM、KNN 与集成方法，附 Python 实现。

!!! note
    建模前务必划分训练集/测试集并做特征标准化。

## SVM 分类示例

```python
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score

# 生成二分类数据
X, y = make_classification(n_samples=200, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)

# 训练 SVM
clf = SVC(kernel="rbf", probability=True)
clf.fit(X_train, y_train)

y_pred = clf.predict(X_test)
print("准确率:", accuracy_score(y_test, y_pred))
```
