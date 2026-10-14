---
title: 大乐透开奖数据爬虫
tags:
  - 项目
  - 爬虫
  - 体彩
---

## 介绍 / Introduction

爬取中国体育彩票大乐透历史开奖号码，存入 CSV/SQLite 用于后续统计分析与选号策略研究。

!!! note
    数据源为体彩官网公开接口，仅用于个人学习分析。

## 抓取示例

```python
import requests
import csv
from datetime import datetime

URL = "https://webapi.sporttery.cn/gateway/lottery/getHistoryPageListV1.qry"

def fetch_draws(page: int = 1, size: int = 30) -> list:
    params = {
        "gameNo": "85",
        "provinceId": "0",
        "pageSize": size,
        "isVerify": "1",
        "pageNo": page,
    }
    resp = requests.get(URL, params=params, timeout=10)
    return resp.json()["value"]["list"]

rows = fetch_draws(page=1, size=50)
for row in rows[:3]:
    print(row["lotteryDrawNum"], row["lotteryDrawResult"])
```

```bash
pip install requests
python dlt_spider.py
```
