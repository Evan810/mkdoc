---
title: Spring Boot 整合 Elasticsearch
tags:
  - 后端
  - Spring Boot
  - 搜索引擎
---

## 介绍 / Introduction

Elasticsearch 是分布式全文搜索引擎，适合商品搜索、日志分析等场景。

!!! note
    推荐使用官方 Java Client（`co.elastic.clients`）替代旧版 RestHighLevelClient。

## 依赖配置

```xml
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>8.12.0</version>
</dependency>
```

## 索引文档示例

```java
@Autowired
private ElasticsearchClient esClient;

public void index(String id, String json) throws IOException {
    esClient.index(i -> i
        .index("products")
        .id(id)
        .document(json)
    );
}
```

```bash
# Docker 启动单节点 ES
docker run -p 9200:9200 -e "discovery.type=single-node" elasticsearch:8.12.0
```
