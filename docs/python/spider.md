---
title: Python 网络爬虫
tags:
  - Python
  - 爬虫
  - requests
---

## 介绍 / Introduction

Python 网络爬虫从入门到进阶，涵盖 requests、BeautifulSoup、XPath、Selenium 与反爬应对。

!!! warning
    请遵守目标网站 robots.txt 及相关法律法规，控制请求频率，不要给目标站点造成压力。

## 安装依赖

```bash
pip install requests beautifulsoup4 lxml selenium aiohttp
```

## requests 基础

```python
import requests

url = "https://example.com"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
}

# GET 请求
resp = requests.get(url, headers=headers, timeout=10)
resp.raise_for_status()          # 非 2xx 抛异常
resp.encoding = resp.apparent_encoding   # 修正中文乱码
print(resp.status_code, resp.text[:100])

# 带查询参数
params = {"q": "python", "page": 1}
requests.get("https://example.com/search", params=params, headers=headers)

# POST 表单 / JSON
requests.post(url, data={"name": "Alice"}, headers=headers)
requests.post(url, json={"name": "Alice"}, headers=headers)

# 会话保持（自动携带 Cookie）
session = requests.Session()
session.headers.update(headers)
session.get(url)
```

## BeautifulSoup 解析

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(resp.text, "lxml")

soup.title.text                     # 页面标题
soup.find("div", class_="item")     # 首个匹配
soup.find_all("a", class_="title")  # 全部匹配
soup.select("ul.list > li a")       # CSS 选择器
soup.select_one("#main .content")   # 单个

# 提取属性与文本
for a in soup.select("a.title"):
    print(a.get("href"), a.get_text(strip=True))
```

## lxml + XPath

```python
from lxml import etree

html = etree.HTML(resp.text)
titles = html.xpath('//div[@class="item"]/a/text()')
links = html.xpath('//a/@href')
# 严格 XPath 提取（返回原字符串，不去空白）
titles = html.xpath('//h2/text()')
```

## 数据存储

```python
import csv
import json

# 存 CSV
with open("data.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["title", "url"])
    writer.writeheader()
    writer.writerows(items)

# 存 JSON
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(items, f, ensure_ascii=False, indent=2)
```

## Selenium 动态渲染

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://example.com")

# 等待元素出现
item = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.CSS_SELECTOR, ".item"))
)
print(item.text)

# 执行 JS / 滚动
driver.execute_script("window.scrollTo(0, document.body.scrollHeight)")
driver.quit()
```

## 异步并发抓取

```python
import asyncio
import aiohttp

async def fetch(session, url):
    async with session.get(url, timeout=10) as resp:
        return await resp.text()

async def main(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, u) for u in urls]
        return await asyncio.gather(*tasks)

asyncio.run(main(["https://example.com"] * 5))
```

## 反爬应对

| 反爬手段 | 应对方式 |
| --- | --- |
| User-Agent 校验 | 设置真实 UA，随机轮换 UA 池 |
| IP 频率限制 | 使用代理池、控制请求间隔 |
| Cookie / 登录态 | 使用 `requests.Session` 保持会话 |
| 动态渲染 | Selenium / Playwright 或分析接口 |
| 参数加密签名 | 逆向 JS，还原加密逻辑 |
| 验证码 | 打码平台或手动处理 |

```python
import time, random

# 随机延时，模拟人工
time.sleep(random.uniform(1, 3))

# 代理
proxies = {"http": "http://127.0.0.1:7890", "https": "http://127.0.0.1:7890"}
requests.get(url, headers=headers, proxies=proxies, timeout=10)
```

!!! tip
    优先分析站点接口（XHR/Fetch）直接请求 JSON 数据，比解析 HTML 更稳定、更高效。
