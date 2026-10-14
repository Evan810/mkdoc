---
title: 数学建模 - 评价模型
tags:
  - 数学建模
  - 评价模型
  - AHP
  - TOPSIS
---

## 介绍 / Introduction

多指标综合评价方法，包括 AHP 层次分析法、TOPSIS 逼近理想解、熵权法等。

!!! note
    AHP 适合主观权重，熵权法适合客观赋权，可组合使用。

## 熵权法 Python 实现

```python
import numpy as np

def entropy_weight(X: np.ndarray) -> np.ndarray:
    """X: (n_samples, n_features) 归一化后的数据"""
    p = X / X.sum(axis=0)
    n = X.shape[0]
    e = -np.sum(p * np.log(p + 1e-10), axis=0) / np.log(n)
    d = 1 - e
    return d / d.sum()

data = np.array([[0.8, 0.6, 0.9],
                 [0.7, 0.8, 0.5],
                 [0.9, 0.7, 0.7]])
print("熵权法权重:", entropy_weight(data))
```
