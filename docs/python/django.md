---
title: Django
tags:
  - Python
  - Django
  - Web 框架
---

## 介绍 / Introduction

Django 是 Python 生态中功能最全的重型 Web 框架，自带 ORM、Admin、认证等组件，适合快速开发内容型站点与后台系统。

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

项目结构：

```text
mysite/
├── manage.py
├── mysite/
│   ├── settings.py     # 配置
│   ├── urls.py         # 根路由
│   └── wsgi.py
└── blog/
    ├── models.py       # 数据模型
    ├── views.py        # 视图
    ├── urls.py         # 应用路由
    └── admin.py        # 后台注册
```

## 配置 settings

```python
# mysite/settings.py
INSTALLED_APPS = [
    # ...
    "blog",
]

LANGUAGE_CODE = "zh-hans"
TIME_ZONE = "Asia/Shanghai"
USE_TZ = True

# 数据库（默认 SQLite，可换 MySQL）
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.mysql",
        "NAME": "mydb",
        "USER": "root",
        "PASSWORD": "password",
        "HOST": "127.0.0.1",
        "PORT": "3306",
    }
}
```

## 模型与 ORM

```python
# blog/models.py
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    published = models.BooleanField(default=True)

    class Meta:
        ordering = ["-created_at"]

    def __str__(self):
        return self.title
```

```bash
python manage.py makemigrations   # 生成迁移
python manage.py migrate          # 应用迁移
```

```python
# 常用查询
Post.objects.all()
Post.objects.filter(published=True)
Post.objects.exclude(published=False)
Post.objects.get(id=1)                       # 不存在抛 DoesNotExist
Post.objects.filter(title__icontains="py")   # 模糊匹配
Post.objects.order_by("-created_at")[:10]
Post.objects.values("id", "title")           # 返回字典
Post.objects.count()

# 关联查询（外键）
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name="comments")

post.comments.all()                          # 反向关联
```

## 视图

```python
# 函数视图
from django.http import JsonResponse
from django.shortcuts import render, get_object_or_404
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published=True)[:10]
    return render(request, "blog/list.html", {"posts": posts})

def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, "blog/detail.html", {"post": post})

# 类视图
from django.views.generic import ListView, DetailView

class PostListView(ListView):
    model = Post
    template_name = "blog/list.html"
    context_object_name = "posts"
    paginate_by = 10
```

## 路由 URL

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = "blog"
urlpatterns = [
    path("", views.PostListView.as_view(), name="list"),
    path("<int:pk>/", views.post_detail, name="detail"),
]
```

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("blog/", include("blog.urls")),
]
```

```python
# 模板中反向解析
# {% url 'blog:detail' post.pk %}
```

## 模板

```html
<!-- blog/templates/blog/list.html -->
{% for post in posts %}
<article>
  <h2><a href="{% url 'blog:detail' post.pk %}">{{ post.title }}</a></h2>
  <p>{{ post.content|truncatechars:100 }}</p>
</article>
{% empty %}
<p>暂无文章</p>
{% endfor %}
```

## Admin 后台

```python
# blog/admin.py
from django.contrib import admin
from .models import Post, Comment

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ("title", "published", "created_at")
    list_filter = ("published",)
    search_fields = ("title",)

admin.site.register(Comment)
```

```bash
python manage.py createsuperuser   # 创建后台管理员
```

## 表单

```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ["title", "content"]
```

## REST API（DRF）

```bash
pip install djangorestframework
```

```python
# settings.py -> INSTALLED_APPS 中加入 "rest_framework"

# blog/serializers.py
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ["id", "title", "content", "created_at"]

# blog/views.py
from rest_framework import viewsets
from .models import Post
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

```python
# urls.py 中注册路由
from rest_framework.routers import DefaultRouter
router = DefaultRouter()
router.register("posts", PostViewSet)
urlpatterns += router.urls
```

!!! tip
    生产环境务必关闭 `DEBUG = True`，并正确配置 `ALLOWED_HOSTS` 与静态文件收集（`collectstatic`）。
