---
title: 数学建模分类
---

## 数学建模 / Mathematical Modeling

数学建模分类汇总数据分析、评价、预测、优化、分类等常见建模方法。

!!! note
    每篇笔记包含算法原理 + Python 实现 + 示例数据。

### 主要方向

- [数据分析](data-analysis.md)
- [评价模型](evaluation.md)
- [预测模型](forecast.md)
- [优化模型](optimization.md)
- [分类模型](classification.md)

```python
import numpy as np

# 简单线性回归示例
x = np.array([1, 2, 3, 4, 5])
y = 2 * x + 1
print(np.polyfit(x, y, 1))
```
