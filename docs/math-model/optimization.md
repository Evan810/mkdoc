---
title: 数学建模 - 优化模型
tags:
  - 数学建模
  - 优化
  - 线性规划
---

## 介绍 / Introduction

优化模型涵盖线性规划、整数规划、非线性规划与启发式算法（遗传算法、模拟退火）。

!!! note
    推荐使用 PuLP / scipy.optimize 快速求解经典问题。

## 线性规划示例（PuLP）

```python
import pulp

# 最大化 3x + 5y, 约束 x + y <= 4, 2x + y <= 6, x,y >= 0
prob = pulp.LpProblem("demo", pulp.LpMaximize)
x = pulp.LpVariable("x", lowBound=0)
y = pulp.LpVariable("y", lowBound=0)

prob += 3 * x + 5 * y
prob += x + y <= 4
prob += 2 * x + y <= 6

prob.solve()
print("状态:", pulp.LpStatus[prob.status])
print(f"x={x.varValue}, y={y.varValue}, 目标值={pulp.value(prob.objective)}")
```

```bash
pip install pulp
```
