---
title: Python 基础
tags:
  - Python
  - 基础语法
---

## 介绍 / Introduction

Python 基础语法速查，涵盖数据类型、控制流、函数、面向对象等核心知识点。

!!! note
    本文档默认 Python 3.10+ 语法，使用了 match-case 等新特性。

## 数据类型速览

```python
# 基本类型
name: str = "Alice"
age: int = 20
height: float = 1.75
is_student: bool = True

# 容器
nums: list[int] = [1, 2, 3]
person: dict[str, object] = {"name": name, "age": age}
unique: set[int] = {1, 2, 3}
point: tuple[int, int] = (10, 20)

print(f"{name} is {age} years old")
```

## 列表推导式

```python
squares = [x * x for x in range(10) if x % 2 == 0]
print(squares)  # [0, 4, 16, 36, 64]
```
