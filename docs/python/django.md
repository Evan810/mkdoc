---
title: Django
tags:
  - Python
  - Django
  - Web 框架
---

## 介绍 / Introduction

Django 是 Python 生态中功能最全的重型 Web 框架，自带 ORM、Admin、认证等组件。

!!! note
    新项目建议搭配 Django REST Framework 构建 API 服务。

## 创建项目

```bash
pip install django
django-admin startproject mysite
cd mysite
python manage.py startapp blog
python manage.py runserver
```

## 视图示例

```python
# blog/views.py
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from .models import Post

def post_list(request):
    posts = Post.objects.all().order_by("-created_at")[:10]
    data = [{"id": p.id, "title": p.title} for p in posts]
    return JsonResponse({"posts": data})
```
