---
title: Python 基础
tags:
  - Python
  - 基础语法
---

## 介绍 / Introduction

Python 基础语法速查，涵盖数据类型、运算符、字符串、容器、控制流、函数、异常、面向对象、模块、文件、类型注解等核心知识点。

!!! note
    本文档默认 Python 3.10+ 语法，使用了 `match-case`、`X | Y` 类型注解等新特性。

## 数据类型速览

```python
# 基本类型
name: str = "Alice"
age: int = 20
height: float = 1.75
is_student: bool = True
nothing: None = None

# 容器
nums: list[int] = [1, 2, 3]                    # 列表：可变、有序
point: tuple[int, int] = (10, 20)              # 元组：不可变、有序
person: dict[str, object] = {"name": name}     # 字典：可变、键值对
unique: set[int] = {1, 2, 3}                   # 集合：可变、去重

# 类型转换
int("42")      # 42
float("3.14")  # 3.14
str(100)       # "100"
list("abc")    # ['a', 'b', 'c']
bool(0)        # False

# 查看类型
type(age)              # <class 'int'>
isinstance(age, int)   # True
```

| 类型 | 可变 | 有序 | 可哈希 | 典型用途 |
| --- | --- | --- | --- | --- |
| `list` | 是 | 是 | 否 | 有序集合、栈/队列 |
| `tuple` | 否 | 是 | 是 | 固定记录、字典键 |
| `dict` | 是 | 是（3.7+） | 否 | 键值映射 |
| `set` | 是 | 否 | 否 | 去重、集合运算 |

## 运算符

```python
# 算术
7 + 2   # 9      7 - 2   # 5
7 * 2   # 14     7 / 2   # 3.5（真除法，结果为 float）
7 // 2  # 3（整除）   7 % 2  # 1（取模）
7 ** 2  # 49（幂）

# 比较 / 逻辑
a, b = 1, 2
a == b   # False      a != b   # True
a < b and b > 0   # True
a > b or b > 0    # True
not a             # False

# 位运算
5 & 3   # 1（与）   5 | 3  # 7（或）
5 ^ 3   # 6（异或）  ~5     # -6（取反）
1 << 3  # 8（左移）  8 >> 2  # 2（右移）

# 成员 / 身份
"a" in "abc"       # True
3 not in [1, 2]    # True
a is None          # False（比较 None 用 is）

# 海象运算符（3.8+）
if (n := len("hello")) > 3:
    print(n)  # 5

# 三元表达式
grade = "及格" if age >= 18 else "未成年"
```

## 字符串常用操作

```python
s = "  Hello, Python  "

s.strip()              # "Hello, Python"（去首尾空白）
s.lower()              # "  hello, python  "
s.upper()              # "  HELLO, PYTHON  "
s.replace("Python", "World")   # 替换
s.split(",")           # ['  Hello', ' Python  ']
"-".join(["a", "b"])   # "a-b"
s.startswith("  He")   # True
s.find("Python")       # 9（找不到返回 -1）
s.count("l")           # 3

# 切片
s2 = "Python"
s2[0]      # 'P'
s2[-1]     # 'n'
s2[1:4]    # 'yth'
s2[::2]    # 'Pto'
s2[::-1]   # 'nohtyP'（反转）

# 格式化
name, age = "Alice", 20
f"{name} is {age} years old"        # f-string（推荐）
"{} is {} years old".format(name, age)
f"{3.14159:.2f}"                    # '3.14'（保留两位小数）
f"{name:>10}"                       # 右对齐宽度 10

# 多行字符串
text = """第一行
第二行"""
```

## 列表 / 元组

```python
nums = [3, 1, 2]

nums.append(4)         # [3, 1, 2, 4]  末尾追加
nums.insert(0, 9)      # [9, 3, 1, 2, 4]  指定位置插入
nums.extend([5, 6])    # 合并另一个可迭代对象
nums.remove(9)         # 删除首个匹配值
popped = nums.pop()    # 弹出末尾元素
nums.index(1)          # 查找下标
nums.count(2)          # 统计出现次数

nums.sort()                    # 原地升序
nums.sort(reverse=True)        # 原地降序
sorted(nums)                   # 返回新列表，不改原列表
nums.reverse()                 # 原地反转

# 切片赋值 / 复制
nums[1:3] = [10, 20]
copy = nums[:]                 # 浅拷贝
length = len(nums)

# 元组：不可变，可用于解包
t = (1, 2, 3)
x, y, z = t                    # 解包
first, *rest = t               # first=1, rest=[2, 3]
```

## 字典

```python
user = {"name": "Alice", "age": 20}

user["age"]                    # 20
user.get("email")              # None（不存在不报错）
user.get("email", "N/A")       # 指定默认值
user.setdefault("city", "武汉") # 不存在则设置并返回
user.keys()                    # 所有键
user.values()                  # 所有值
user.items()                   # 键值对

user.update({"age": 21, "job": "dev"})   # 批量更新
user.pop("job")                          # 删除并返回
"name" in user                           # True（判断键是否存在）

# 遍历
for k, v in user.items():
    print(k, v)

# 字典推导式
squares = {x: x * x for x in range(5)}

# 合并（3.9+）
config = {**user, "theme": "dark"}

# 有序计数
from collections import Counter
Counter("hello")   # Counter({'l': 2, 'h': 1, 'e': 1, 'o': 1})
```

## 集合

```python
a = {1, 2, 3}
b = {3, 4, 5}

a | b    # {1, 2, 3, 4, 5}  并集
a & b    # {3}              交集
a - b    # {1, 2}           差集
a ^ b    # {1, 2, 4, 5}     对称差

a.add(9)          # 添加元素
a.discard(9)      # 删除（不存在不报错）
a.remove(99)      # 删除（不存在报 KeyError）

# 集合推导式
{x % 3 for x in range(10)}

# 去重（不保留顺序）
list(set([1, 2, 2, 3]))   # [1, 2, 3]
```

## 条件与循环

```python
# if / elif / else
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
else:
    grade = "C"

# for 循环
for i in range(3):          # 0 1 2
    print(i)

for i, item in enumerate(["a", "b"]):   # 带下标
    print(i, item)

for a, b in zip([1, 2], ["x", "y"]):    # 并行遍历
    print(a, b)

# while 循环
n = 0
while n < 3:
    n += 1

# break / continue / else
for x in range(5):
    if x == 3:
        break        # 跳出整个循环
    if x == 1:
        continue     # 跳过本次
else:
    print("循环正常结束才执行")   # 被 break 则不执行

# match-case（3.10+）
def http_status(code: int) -> str:
    match code:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case _:
            return "Unknown"
```

## 推导式

```python
# 列表推导式
squares = [x * x for x in range(10) if x % 2 == 0]
print(squares)  # [0, 4, 16, 36, 64]

# 嵌套推导式
matrix = [[i * j for j in range(3)] for i in range(3)]

# 字典推导式
word_len = {w: len(w) for w in ["hi", "hello"]}   # {'hi': 2, 'hello': 5}

# 集合推导式
evens = {x for x in range(10) if x % 2 == 0}

# 生成器表达式（惰性求值，省内存）
total = sum(x * x for x in range(1000000))
```

## 函数

```python
def greet(name: str, greeting: str = "Hello") -> str:
    """返回问候语。"""
    return f"{greeting}, {name}!"

greet("Alice")                  # 'Hello, Alice!'
greet("Bob", greeting="Hi")     # 关键字参数
greet(greeting="Yo", name="C")  # 顺序无关

# 可变参数
def total(*args, **kwargs):
    print(args)      # 位置参数元组
    print(kwargs)    # 关键字参数字典

total(1, 2, x=3)

# lambda（匿名函数）
add = lambda a, b: a + b
sorted(["bb", "a"], key=lambda s: len(s))   # ['a', 'bb']

# 装饰器
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"调用 {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def hello():
    print("hi")

# 闭包 / 作用域
def counter():
    count = 0
    def inc():
        nonlocal count   # 修改外层变量
        count += 1
        return count
    return inc
```

## 异常处理

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"捕获异常：{e}")
except (ValueError, TypeError):
    print("多种异常")
else:
    print("无异常时执行")
finally:
    print("无论如何都执行")

# 主动抛出
def check(age: int) -> None:
    if age < 0:
        raise ValueError("年龄不能为负")

# 自定义异常
class MyError(Exception):
    pass

# 上下文管理器式忽略异常
from contextlib import suppress
with suppress(FileNotFoundError):
    open("not_exist.txt")
```

## 类与对象

```python
class Animal:
    species = "动物"              # 类属性

    def __init__(self, name: str):
        self.name = name         # 实例属性

    def speak(self) -> str:      # 实例方法
        return f"{self.name} 发出声音"

    @classmethod
    def create(cls, name: str):  # 类方法
        return cls(name)

    @staticmethod
    def info() -> str:           # 静态方法
        return "这是一个类"

class Dog(Animal):               # 继承
    def speak(self) -> str:      # 重写
        return f"{self.name} 汪汪叫"

    def __str__(self) -> str:    # 魔术方法
        return f"Dog({self.name})"

d = Dog("旺财")
d.speak()            # '旺财 汪汪叫'
isinstance(d, Animal)  # True

# 数据类（3.7+）
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

p = Point(1, 2)      # 自动生成 __init__/__repr__/__eq__
```

## 模块与包

```python
# 导入方式
import math
from math import sqrt, pi
from math import sqrt as sq
from collections import defaultdict

# 常用写法
if __name__ == "__main__":
    main()

# 包结构
# mypkg/
#   __init__.py
#   module_a.py
#   sub/
#     __init__.py
#     module_b.py

from mypkg import module_a
from mypkg.sub import module_b
```

## 文件操作

```python
# 读取（推荐 with，自动关闭）
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()            # 读取全部
    # lines = f.readlines()       # 按行读取为列表

with open("data.txt", encoding="utf-8") as f:
    for line in f:                # 逐行读取，省内存
        print(line.rstrip())

# 写入
with open("out.txt", "w", encoding="utf-8") as f:
    f.write("hello\n")

# 追加
with open("out.txt", "a", encoding="utf-8") as f:
    f.write("world\n")

# 模式：r 读 / w 覆盖写 / a 追加 / b 二进制 / r+ 读写

# 路径操作（推荐 pathlib）
from pathlib import Path

p = Path("docs") / "index.md"
p.exists()          # 是否存在
p.suffix            # '.md'
p.stem              # 'index'
p.read_text(encoding="utf-8")
```

## 常用内置函数

```python
len([1, 2, 3])            # 3
sum([1, 2, 3])            # 6
max([1, 5, 2])            # 5
min([1, 5, 2])            # 1
abs(-3)                   # 3
round(3.567, 1)           # 3.6
sorted([3, 1, 2])         # [1, 2, 3]
reversed([1, 2, 3])       # 反向迭代器
enumerate(["a", "b"])     # 带下标的迭代器
zip([1, 2], ["a", "b"])   # 打包迭代
map(lambda x: x * 2, [1, 2])   # 映射
filter(lambda x: x > 1, [1, 2])  # 过滤
any([False, True])        # True
all([True, True])         # True
isinstance(1, int)        # True
```

## 上下文管理器

```python
# 自定义上下文管理器
class Timer:
    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        print(f"耗时 {time.perf_counter() - self.start:.3f}s")
        return False   # 返回 True 可吞掉异常

with Timer():
    sum(range(1000000))

# 用装饰器简化
from contextlib import contextmanager

@contextmanager
def opened(path: str):
    print("打开")
    yield path
    print("关闭")

with opened("file.txt") as p:
    print(p)
```

## 类型注解

```python
from typing import Optional, Union, Any, Callable
from collections.abc import Iterable

# 变量与函数注解
count: int = 0
names: list[str] = ["a"]

def process(data: dict[str, int], flag: bool = False) -> list[int]:
    return list(data.values())

# 3.10+ 可用 X | Y 简写
def find(key: str) -> int | None:
    ...

# 可选 / 联合
value: Optional[str] = None          # 等价于 str | None
number: Union[int, float] = 1.0      # 等价于 int | float

# 可调用与任意类型
callback: Callable[[int], str] = str
anything: Any = object()

# 类型别名
Vector = list[float]

# 运行时不强制，需 mypy / pyright 等工具检查
```

## 常用标准库

```python
# 路径与系统
from pathlib import Path
import os, sys

# 时间
from datetime import datetime, timedelta
datetime.now()                       # 当前时间
datetime.now() + timedelta(days=1)   # 明天

# 随机
import random
random.randint(1, 10)                # 随机整数
random.choice(["a", "b"])            # 随机元素

# 正则
import re
re.findall(r"\d+", "a1b22")          # ['1', '22']
re.sub(r"\d", "*", "a1b2")           # 'a*b*'

# JSON
import json
json.dumps({"a": 1})                 # 序列化为字符串
json.loads('{"a": 1}')               # 反序列化

# 常用容器
from collections import defaultdict, Counter, deque, namedtuple

d = defaultdict(list)                # 默认值字典
d["k"].append(1)

q = deque([1, 2])                    # 双端队列
q.appendleft(0)

Point = namedtuple("Point", ["x", "y"])   # 命名元组
p = Point(1, 2)
p.x
```

!!! tip
    速查建议：日常优先使用 `f-string`、列表/字典推导式、`pathlib`、`with` 上下文管理器与类型注解，代码更简洁、更安全。
