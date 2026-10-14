---
title: 数学建模 - 数据分析
tags:
  - 数学建模
  - 数据分析
  - Python
---

## 介绍 / Introduction

数学建模中的数据分析方法，涵盖描述统计、相关性分析、主成分分析（PCA）等。

!!! note
    所有示例数据均使用 numpy / pandas 生成或加载。

## PCA 降维示例

```python
import numpy as np
from sklearn.decomposition import PCA

# 生成 3 维数据
rng = np.random.default_rng(42)
X = rng.normal(size=(100, 3))

# 降到 2 维
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)

print("解释方差比:", pca.explained_variance_ratio_)
print("降维后 shape:", X_reduced.shape)
```

```bash
pip install scikit-learn
```
