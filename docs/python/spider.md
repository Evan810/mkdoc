---
title: Python 网络爬虫
tags:
  - Python
  - 爬虫
  - requests
---

## 介绍 / Introduction

Python 网络爬虫从入门到进阶，涵盖 requests、BeautifulSoup、Selenium 与反爬应对。

!!! warning
    请遵守目标网站 robots.txt 及相关法律法规，控制请求频率。

## 基础 requests 使用

```python
import requests
from bs4 import BeautifulSoup

url = "https://example.com"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
}

resp = requests.get(url, headers=headers, timeout=10)
resp.raise_for_status()

soup = BeautifulSoup(resp.text, "html.parser")
titles = [a.get_text(strip=True) for a in soup.select("a.title")]
print(titles)
```

## 安装依赖

```bash
pip install requests beautifulsoup4 lxml
```
