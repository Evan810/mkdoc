---
title: Spring Boot 整合 Elasticsearch
tags:
  - 后端
  - Spring Boot
  - 搜索引擎
  - Elasticsearch
---

## 介绍 / Introduction

Elasticsearch（ES）是分布式全文搜索引擎，基于 Lucene 构建，适合商品搜索、日志分析、日志检索等场景。本文覆盖核心概念、REST API 速查、DSL 查询、分词器、聚合，以及 Spring Boot 的两条整合路线。

!!! note
    推荐使用官方 Java Client（`co.elastic.clients`）替代旧版 `RestHighLevelClient`。
    本文示例基于 ES 8.x + Spring Boot 3.x。

## 核心概念 / Concepts

| 概念 | 说明 | 类比 MySQL |
| --- | --- | --- |
| Index | 索引，一类文档的集合 | 数据库 / 表 |
| Document | 文档，最小数据单元（JSON） | 一行记录 |
| Field | 字段 | 列 |
| Mapping | 字段类型与分词定义 | 表结构 |
| Shard | 分片，索引的水平切分，实现分布式 | 分库分表 |
| Replica | 副本，分片的冗余，提供高可用与读扩展 | 从库 |
| Inverted Index | 倒排索引，词 → 文档 ID 列表 | 全文索引 |

!!! note
    倒排索引：先对文档内容分词，建立「词项（term）→ 包含该词的文档 ID 列表」的映射，因此能按词快速定位文档。ES 默认对 `text` 字段分词，对 `keyword` 字段不分词（整体精确匹配）。

## 安装与启动 / Install & Docker

```bash
# 单节点启动（8.x 默认开启安全认证，开发环境可关闭）
docker run -d --name es \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  elasticsearch:8.12.0

# 验证：curl http://localhost:9200
```

```bash
# 可选：启动 Kibana（可视化 + Dev Tools）
docker run -d --name kibana \
  -p 5601:5601 \
  -e "ELASTICSEARCH_HOSTS=http://host.docker.internal:9200" \
  kibana:8.12.0
# 访问 http://localhost:5601 → Dev Tools 执行 DSL
```

!!! warning
    生产环境务必开启 `xpack.security`，不要用 `security.enabled=false`。开发环境关闭仅为方便调试。

## REST API 速查 / REST

### 索引 CRUD

```bash
# 创建索引（可省略 body，ES 会动态映射）
curl -X PUT "localhost:9200/products"

# 创建索引并指定 mapping 与分片
curl -X PUT "localhost:9200/products" -H 'Content-Type: application/json' -d'
{
  "settings": { "number_of_shards": 3, "number_of_replicas": 1 },
  "mappings": {
    "properties": {
      "title":     { "type": "text", "analyzer": "ik_max_word" },
      "brand":     { "type": "keyword" },
      "price":     { "type": "double" },
      "createdAt": { "type": "date" }
    }
  }
}'

# 查看 / 删除索引
curl "localhost:9200/products/_mapping"
curl -X DELETE "localhost:9200/products"

# 重建索引（修改 mapping 后迁移数据）
curl -X POST "localhost:9200/_reindex" -H 'Content-Type: application/json' -d'
{ "source": { "index": "products_v1" }, "dest": { "index": "products_v2" } }'
```

### 文档 CRUD

```bash
# 新增/全量替换（指定 id）
curl -X PUT "localhost:9200/products/_doc/1" -H 'Content-Type: application/json' -d'
{ "title": "小米手机", "brand": "xiaomi", "price": 2999 }'

# 新增（自动生成 id）
curl -X POST "localhost:9200/products/_doc" -H 'Content-Type: application/json' -d'
{ "title": "华为手机", "brand": "huawei", "price": 4999 }'

# 局部更新
curl -X POST "localhost:9200/products/_update/1" -H 'Content-Type: application/json' -d'
{ "doc": { "price": 2799 } }'

# 查询 / 删除
curl "localhost:9200/products/_doc/1"
curl -X DELETE "localhost:9200/products/_doc/1"
```

### 批量操作 _bulk

```bash
# 每行以换行分隔，action 行与 source 行成对出现
curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' -d'
{ "index": { "_index": "products", "_id": "1" } }
{ "title": "小米手机", "brand": "xiaomi", "price": 2999 }
{ "index": { "_index": "products", "_id": "2" } }
{ "title": "华为手机", "brand": "huawei", "price": 4999 }
{ "update": { "_index": "products", "_id": "3" } }
{ "doc": { "price": 3999 } }
{ "delete": { "_index": "products", "_id": "4" } }
'
```

!!! tip
    批量导入务必使用 `_bulk`（单次可含数千条），比逐条请求快一个数量级。文件末尾需有换行。

## DSL 查询 / Query DSL

### match / term / range

```json
// match：分词后匹配，用于 text 字段
GET /products/_search
{ "query": { "match": { "title": "小米 手机" } } }

// match_phrase：短语匹配，词序必须一致
GET /products/_search
{ "query": { "match_phrase": { "title": "小米手机" } } }

// term：不分词精确匹配，用于 keyword 字段
GET /products/_search
{ "query": { "term": { "brand": "xiaomi" } } }

// range：范围查询
GET /products/_search
{ "query": { "range": { "price": { "gte": 1000, "lte": 5000 } } } }
```

!!! warning
    `term` 不会对查询词分词，对 `text` 字段使用可能导致查不到结果。精确匹配应使用 `keyword` 子字段或先 `analyze`。

### bool 组合查询

```json
GET /products/_search
{
  "query": {
    "bool": {
      "must":     [ { "match": { "title": "手机" } } ],
      "should":   [ { "term":  { "brand": "xiaomi" } },
                    { "term":  { "brand": "huawei" } } ],
      "must_not": [ { "term":  { "status": "deleted" } } ],
      "filter":   [ { "range": { "price": { "gte": 1000 } } } ],
      "minimum_should_match": 1
    }
  }
}
```

| 子句 | 作用 | 参与打分 |
| --- | --- | --- |
| must | 必须匹配（AND） | 是 |
| should | 应该匹配（OR），满足 `minimum_should_match` | 是 |
| must_not | 必须不匹配（NOT） | 否 |
| filter | 过滤，不参与打分，可缓存、更快 | 否 |

!!! tip
    不需要相关度打分的条件（如价格区间、状态）尽量放进 `filter`，性能更好。

### 分页、排序、高亮

```json
GET /products/_search
{
  "from": 0, "size": 10,
  "query": { "match": { "title": "手机" } },
  "sort": [ { "price": { "order": "asc" } }, { "_score": { "order": "desc" } } ],
  "highlight": {
    "pre_tags": ["<em>"], "post_tags": ["</em>"],
    "fields": { "title": {} }
  },
  "_source": ["title", "brand", "price"]
}
```

!!! note
    深分页（`from + size` 过大）性能差，ES 默认 `index.max_result_window=10000`。深翻页用 `search_after`（配合 `sort`）或 `scroll`（导出场景）。

## 分词器 / Analyzer

ES 内置 `standard` / `simple` / `whitespace` 等分词器，对中文支持差，需安装 IK 中文分词插件。

```bash
# 安装 IK 分词器（版本必须与 ES 一致）
docker exec -it es bin/elasticsearch-plugin install \
  https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.12.0/elasticsearch-analysis-ik-8.12.0.zip
docker restart es
```

```bash
# 测试分词效果
GET /_analyze
{ "analyzer": "ik_max_word", "text": "中华人民共和国国歌" }
```

| 分词器 | 特点 |
| --- | --- |
| ik_max_word | 最细粒度拆分，词多，适合索引（召回高） |
| ik_smart | 粗粒度拆分，词少，适合搜索（精确） |

```json
// 索引时用 ik_max_word，搜索时用 ik_smart
"mappings": {
  "properties": {
    "title": { "type": "text", "analyzer": "ik_max_word", "search_analyzer": "ik_smart" }
  }
}
```

## 聚合 / Aggregation

```json
GET /products/_search
{
  "size": 0,
  "aggs": {
    "brand_count": {
      "terms": { "field": "brand", "size": 10 }        // 按品牌分桶统计
    },
    "avg_price": {
      "avg": { "field": "price" }                       // 平均价格
    },
    "price_stats": {
      "stats": { "field": "price" }                     // min/max/avg/sum/count
    },
    "sales_by_month": {
      "date_histogram": {
        "field": "createdAt", "calendar_interval": "month"
      }
    }
  }
}
```

| 聚合类型 | 用途 |
| --- | --- |
| terms | 按字段值分组（类似 GROUP BY） |
| avg / sum / min / max / stats | 数值指标聚合 |
| date_histogram | 按时间区间分桶（趋势图） |
| cardinality | 去重计数（类似 COUNT DISTINCT） |

!!! tip
    聚合字段若是 `text` 类型无法聚合，需使用其 `keyword` 子字段（`brand.keyword`）或 fielddata。

## Spring Boot 整合 / Integration

### 路线一：Spring Data Elasticsearch

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

```yaml
spring:
  elasticsearch:
    uris: http://127.0.0.1:9200
    connection-timeout: 3s
    socket-timeout: 30s
```

```java
/** 实体：映射到索引 products */
@Document(indexName = "products")
public class Product {
    @Id
    private String id;
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String title;
    @Field(type = FieldType.Keyword)
    private String brand;
    @Field(type = FieldType.Double)
    private Double price;
    // getter / setter 省略
}
```

```java
/** Repository：方法名自动生成查询 */
public interface ProductRepository extends ElasticsearchRepository<Product, String> {
    List<Product> findByBrand(String brand);
    List<Product> findByTitleContaining(String keyword);
    List<Product> findByPriceBetween(Double min, Double max);
}
```

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository repository;

    public void save(Product p) { repository.save(p); }

    public List<Product> search(String kw) {
        // repository 支持分页、排序等
        return repository.findByTitleContaining(kw);
    }
}
```

### 路线二：官方 Java Client

```xml
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>8.12.0</version>
</dependency>
```

```java
@Configuration
public class EsConfig {

    @Bean
    public ElasticsearchClient elasticsearchClient() {
        RestClient restClient = RestClient.builder(
                new HttpHost("127.0.0.1", 9200, "http")).build();
        ElasticsearchTransport transport = new RestClientTransport(
                restClient, new JacksonJsonpMapper());
        return new ElasticsearchClient(transport);
    }
}
```

```java
@Service
public class ProductEsService {

    @Autowired
    private ElasticsearchClient esClient;

    /** 索引文档 */
    public void index(String id, Product product) throws IOException {
        esClient.index(i -> i.index("products").id(id).document(product));
    }

    /** 条件搜索：title 分词匹配 + 价格过滤，返回高分排序 */
    public List<Product> search(String keyword, double minPrice) throws IOException {
        SearchResponse<Product> resp = esClient.search(s -> s
                        .index("products")
                        .query(q -> q.bool(b -> b
                                .must(m -> m.match(mt -> mt.field("title").query(keyword)))
                                .filter(f -> f.range(r -> r.field("price")
                                        .gte(JsonData.of(minPrice))))))
                        .sort(so -> so.field(f -> f.field("_score").order(SortOrder.Desc))),
                Product.class);
        return resp.hits().hits().stream().map(Hit::source).toList();
    }
}
```

### 两条路线对比

| 维度 | Spring Data ES | 官方 Java Client |
| --- | --- | --- |
| 抽象层次 | 高，Repository 风格 | 低，贴近原生 API |
| 学习成本 | 低 | 中 |
| 复杂 DSL | 需 NativeQuery | 灵活、完整 |
| 适用场景 | 简单 CRUD / 查询 | 复杂聚合、精细控制 |

## 常见业务场景 / Use Cases

### 商品搜索

- 关键词：`title` 用 `ik_max_word` 建索引、`ik_smart` 搜索。
- 过滤：品牌 / 分类 / 价格区间放 `filter`（不打分、可缓存）。
- 排序：销量、价格、相关度。
- 高亮：命中关键词高亮返回前端。
- 分页：浅分页用 `from/size`，深翻页用 `search_after`。

### 日志分析

- 按天滚动索引：`logs-2026.10.18`，配合 ILM 生命周期管理自动删除旧日志。
- 常用聚合：`date_histogram` 画趋势、`terms` 统计 TopN 错误码。
- 配套：Logstash / Filebeat 采集，Kibana 可视化；或直接写 ES。

## 最佳实践 / Best Practices

| 主题 | 建议 |
| --- | --- |
| Mapping | 提前定义，字段类型一旦创建不可修改，变更需 reindex |
| 字段类型 | 全文检索用 text，精确匹配/聚合用 keyword |
| 分片 | 单分片建议 10–50GB，分片过多影响集群稳定性 |
| 副本 | 至少 1 个副本保证高可用，写入密集时先设 0 再改回 |
| 批量导入 | 使用 `_bulk`，控制单批大小（约 5–15MB） |
| 深分页 | 避免超大 `from`，用 `search_after` / `scroll` |
| 查询 | 不需要打分的条件放 `filter` |
| 别名 | 使用 alias 平滑切换索引版本（reindex 后切别名） |

!!! warning
    不要用 ES 当「主数据库」：ES 无事务、近实时（默认 1s refresh），写入可能丢失。应将 MySQL 作为数据源，ES 作为搜索/分析副本，并做好数据同步。

```json
// 别名平滑迁移
POST /_aliases
{
  "actions": [
    { "remove": { "index": "products_v1", "alias": "products" } },
    { "add":    { "index": "products_v2", "alias": "products" } }
  ]
}
```
