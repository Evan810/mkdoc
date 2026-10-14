---
title: Bilibili UP 主视频爬虫
tags:
  - 项目
  - 爬虫
  - Bilibili
---

## 介绍 / Introduction

抓取 Bilibili 指定 UP 主的视频列表、播放量、弹幕等信息，持久化到数据库并可视化分析。

!!! warning
    请控制请求频率，建议添加随机延时与 Cookie，避免被风控。

## 核心请求示例

```python
import requests

def fetch_up_videos(mid: str, ps: int = 30) -> dict:
    url = "https://api.bilibili.com/x/space/wbi/arc/search"
    params = {"mid": mid, "ps": ps, "pn": 1}
    headers = {
        "User-Agent": "Mozilla/5.0",
        "Referer": f"https://space.bilibili.com/{mid}",
    }
    resp = requests.get(url, params=params, headers=headers)
    resp.raise_for_status()
    return resp.json()

data = fetch_up_videos("123456")
print(data["data"]["list"]["vlist"][0]["title"])
```

```bash
# 运行
pip install requests
python main.py
```
