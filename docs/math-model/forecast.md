---
title: 数学建模 - 预测模型
tags:
  - 数学建模
  - 预测
  - ARIMA
  - 回归
---

## 介绍 / Introduction

预测模型涵盖时间序列（ARIMA、Prophet）与回归预测（线性回归、随机森林等）。

!!! note
    时间序列数据需先进行平稳性检验（ADF 检验）。

## ARIMA 示例

```python
import numpy as np
from statsmodels.tsa.arima.model import ARIMA

# 生成趋势数据
rng = np.random.default_rng(0)
series = np.cumsum(rng.normal(0, 1, 100)) + np.linspace(0, 10, 100)

# 拟合 ARIMA(1,1,1)
model = ARIMA(series, order=(1, 1, 1)).fit()

# 预测未来 10 步
forecast = model.forecast(steps=10)
print(forecast)
```

```bash
pip install statsmodels
```
