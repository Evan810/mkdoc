---
title: Spring Boot 技术整合
tags:
  - 后端
  - Spring Boot
  - 微服务
  - Redis
  - Nacos
  - Sentinel
  - Seata
---

## 介绍 / Introduction

记录 Spring Boot 与常见中间件（Redis、Nacos、Sentinel、Seata 等）的整合实战，偏「速查 + 实战」，代码可直接复制修改使用。

!!! note
    所有示例基于 Spring Boot 3.x + Java 17 + Spring Cloud 2023.x（对应 Spring Cloud Alibaba 2023.x）。
    注意 Spring Boot 3 已全面迁移到 `jakarta.*` 命名空间，不再使用 `javax.*`。

!!! tip
    Spring Cloud 与 Spring Boot 的版本必须严格对齐官方兼容矩阵，否则启动阶段就会因 `spring-cloud-context` 版本冲突报错。

---

## Spring Boot 核心 / Core

### starter 机制

Starter 本质是一个「依赖 + 自动配置」的组合包：引入一个 starter，Maven 会连带把所需的库拉齐，同时通过 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 注册自动配置类。

| Starter | 作用 |
| --- | --- |
| `spring-boot-starter-web` | Web / REST（Tomcat + Jackson） |
| `spring-boot-starter-data-redis` | Redis（Lettuce + StringRedisTemplate） |
| `spring-boot-starter-validation` | 参数校验（Hibernate Validator） |
| `spring-boot-starter-aop` | AOP 切面支持 |
| `spring-boot-starter-actuator` | 健康检查 / 监控端点 |

```xml
<!-- 自定义 starter 的自动配置注册文件（Spring Boot 3 路径） -->
<!-- 文件位置：src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports -->
<!-- 内容示例：每行一个全限定类名 -->
com.example.demo.config.MyAutoConfiguration
```

### 自动配置原理

```java
// 自定义自动配置类：满足条件时才生效
@AutoConfiguration
@ConditionalOnClass(MyService.class)          // 类路径存在才装配
@ConditionalOnProperty(prefix = "my", name = "enabled", havingValue = "true", matchIfMissing = true)
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean                 // 用户未自定义时才用默认实现
    public MyService myService(MyProperties props) {
        return new MyService(props);
    }
}
```

!!! note
    `@ConditionalOnMissingBean` 是自动配置「可被用户覆盖」的关键，脚手架默认实现务必加上它。

### 注解速查表

| 注解 | 说明 |
| --- | --- |
| `@SpringBootApplication` | 组合注解 = `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| `@Configuration` | 声明配置类（Spring Boot 3 默认 proxyBeanMethods=true） |
| `@Bean` | 将方法返回值注册为 Bean |
| `@Autowired` | 按类型注入（推荐构造器注入） |
| `@Component` / `@Service` / `@Repository` / `@Controller` | 分层组件声明 |
| `@RestController` | `@Controller` + `@ResponseBody` |
| `@ConfigurationProperties` | 批量绑定配置文件属性到对象 |
| `@Value` | 注入单个配置值（`${key:default}`） |
| `@Profile` | 按环境激活 Bean |
| `@ConditionalOnXxx` | 条件装配 |
| `@PostConstruct` / `@PreDestroy` | 生命周期回调 |

```java
// 推荐：构造器注入 + ConfigurationProperties 绑定
@Service
@RequiredArgsConstructor                          // Lombok：为 final 字段生成构造器
public class OrderService {
    private final OrderRepository orderRepository;
}
```

### application.yml 常用配置

```yaml
server:
  port: 8080
  servlet:
    context-path: /api
  tomcat:
    threads:
      max: 200
  shutdown: graceful          # 优雅停机（见后文）

spring:
  application:
    name: demo-service
  profiles:
    active: dev
  lifecycle:
    timeout-per-shutdown-phase: 30s   # 优雅停机等待时长
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/demo?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  data:
    redis:
      host: 127.0.0.1
      port: 6379
      database: 0
      timeout: 3s
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

logging:
  level:
    root: info
    com.example.demo: debug
```

!!! warning
    配置文件支持分环境拆分：`application-dev.yml` / `application-prod.yml`，用 `spring.profiles.active` 或启动参数 `--spring.profiles.active=prod` 切换。

---

## 统一响应与全局异常处理 / Unified Response & Global Exception

### 统一响应体

```java
import lombok.Data;

@Data
public class Result<T> {
    private int code;          // 0=成功，其他=业务错误码
    private String message;
    private T data;

    public static <T> Result<T> ok(T data) {
        Result<T> r = new Result<>();
        r.setCode(0);
        r.setMessage("success");
        r.setData(data);
        return r;
    }

    public static <T> Result<T> fail(int code, String message) {
        Result<T> r = new Result<>();
        r.setCode(code);
        r.setMessage(message);
        return r;
    }
}
```

### 自定义业务异常

```java
public class BizException extends RuntimeException {
    private final int code;

    public BizException(String message) {
        this(500, message);
    }

    public BizException(int code, String message) {
        super(message);
        this.code = code;
    }

    public int getCode() {
        return code;
    }
}
```

### 全局异常处理

```java
import jakarta.validation.ConstraintViolationException;   // 注意 jakarta，不是 javax
import org.springframework.validation.BindException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    /** 业务异常 */
    @ExceptionHandler(BizException.class)
    public Result<Void> handleBiz(BizException e) {
        return Result.fail(e.getCode(), e.getMessage());
    }

    /** @RequestBody 参数校验失败 */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValid(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldErrors().stream()
                .map(f -> f.getField() + ": " + f.getDefaultMessage())
                .findFirst().orElse("参数校验失败");
        return Result.fail(400, msg);
    }

    /** 表单/查询参数绑定失败 */
    @ExceptionHandler(BindException.class)
    public Result<Void> handleBind(BindException e) {
        String msg = e.getBindingResult().getFieldErrors().stream()
                .map(f -> f.getField() + ": " + f.getDefaultMessage())
                .findFirst().orElse("参数绑定失败");
        return Result.fail(400, msg);
    }

    /** @RequestParam / @PathVariable 校验失败 */
    @ExceptionHandler(ConstraintViolationException.class)
    public Result<Void> handleConstraint(ConstraintViolationException e) {
        return Result.fail(400, e.getMessage());
    }

    /** 兜底 */
    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        // 生产环境务必打印日志，不要直接返回堆栈
        return Result.fail(500, "服务器内部错误");
    }
}
```

!!! tip
    兜底异常处理里不要 `return e.getMessage()`，避免把 SQL、路径等敏感信息暴露给前端。

---

## 拦截器 / 过滤器 / 跨域 / Interceptor · Filter · CORS

### 拦截器（HandlerInterceptor）

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        String token = request.getHeader("Authorization");
        if (token == null || token.isBlank()) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;                      // 中断请求
        }
        return true;                           // 放行
    }
}
```

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final AuthInterceptor authInterceptor;

    public WebConfig(AuthInterceptor authInterceptor) {
        this.authInterceptor = authInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/login", "/doc.html", "/swagger-ui/**", "/v3/api-docs/**");
    }
}
```

### 过滤器（Filter）

```java
@Component
@Order(1)                                  // 数值越小优先级越高
public class TraceFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        String traceId = UUID.randomUUID().toString().replace("-", "");
        request.setAttribute("traceId", traceId);
        // MDC 便于日志串联（见后文 logback 配置）
        MDC.put("traceId", traceId);
        try {
            chain.doFilter(req, resp);
        } finally {
            MDC.remove("traceId");
        }
    }
}
```

| 对比项 | Filter | Interceptor |
| --- | --- | --- |
| 规范 | Servlet 规范 | Spring MVC |
| 作用范围 | 所有请求（含静态资源） | 仅被 DispatcherServlet 处理的请求 |
| 获取 Bean | 需注入 | 直接注入 |
| 典型用途 | 编码、TraceId、XSS 过滤 | 鉴权、日志、限流 |

### 跨域 CORS

```java
// 方式一：全局配置（推荐）
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")        // Spring Boot 3 用 patterns 支持带凭据的 *
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

```java
// 方式二：单个接口
@RestController
public class DemoController {
    @CrossOrigin(origins = "http://localhost:5173")
    @GetMapping("/demo")
    public Result<String> demo() {
        return Result.ok("ok");
    }
}
```

!!! warning
    `allowedOrigins("*")` 与 `allowCredentials(true)` 同时开启会被浏览器拒绝，必须改用 `allowedOriginPatterns("*")`。

---

## Redis 整合 / Redis Integration

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- 连接池（可选，Lettuce 默认不启用池化） -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

### 配置

```yaml
spring:
  data:
    redis:
      host: 127.0.0.1
      port: 6379
      password: ""
      database: 0
      timeout: 3s
      lettuce:
        pool:
          max-active: 16
          max-idle: 8
          min-idle: 2
          max-wait: 2s
```

### StringRedisTemplate 常用操作

```java
@Service
public class UserService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    public void cacheUser(Long id, String name) {
        // 设置带过期的字符串
        redisTemplate.opsForValue().set("user:" + id, name, 1, TimeUnit.HOURS);
    }

    public String getUser(Long id) {
        return redisTemplate.opsForValue().get("user:" + id);
    }

    public void opsDemo() {
        // Hash
        redisTemplate.opsForHash().put("user:1", "name", "Alice");
        // List
        redisTemplate.opsForList().rightPush("queue", "task");
        // Set
        redisTemplate.opsForSet().add("tags", "java", "spring");
        // 原子自增（计数器 / 限流）
        redisTemplate.opsForValue().increment("counter");
        // 删除
        redisTemplate.delete("user:1");
    }
}
```

!!! note
    统一用 `StringRedisTemplate`（key/value 均为 String）可读性好；若需存对象，建议手动 JSON 序列化，避免默认的 JDK 序列化产生乱码。

### 缓存注解

```java
@Service
public class ProductService {

    /** 查询：命中则直接返回缓存；否则执行方法并写入缓存（cacheNames + key） */
    @Cacheable(cacheNames = "product", key = "#id", unless = "#result == null")
    public Product getById(Long id) {
        return productMapper.selectById(id);
    }

    /** 更新：清除指定 key，保证一致性 */
    @CacheEvict(cacheNames = "product", key = "#product.id")
    public void update(Product product) {
        productMapper.updateById(product);
    }

    /** 清空整个缓存分区 */
    @CacheEvict(cacheNames = "product", allEntries = true)
    public void refreshAll() {
        // ...
    }
}
```

```java
// 开启缓存注解 + 自定义序列化
@EnableCaching
@Configuration
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30))                 // 默认过期时间
                .serializeKeysWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .disableCachingNullValues();
        return RedisCacheManager.builder(factory).cacheDefaults(config).build();
    }
}
```

!!! warning
    `@Cacheable` 基于代理生效：**同类内部自调用不会走缓存**，且方法必须是 public。

### 分布式锁思路

```java
// 思路：SET NX + 过期时间；释放时用 Lua 校验持有者，防止误删
public boolean tryLock(String key, String requestId, long expireSeconds) {
    Boolean ok = redisTemplate.opsForValue().setIfAbsent(key, requestId, expireSeconds, TimeUnit.SECONDS);
    return Boolean.TRUE.equals(ok);
}

private static final String UNLOCK_LUA =
        "if redis.call('get', KEYS[1]) == ARGV[1] then " +
        "  return redis.call('del', KEYS[1]) else return 0 end";

public void unlock(String key, String requestId) {
    redisTemplate.execute(new DefaultRedisScript<>(UNLOCK_LUA, Long.class),
            Collections.singletonList(key), requestId);
}
```

!!! tip
    生产环境建议直接使用 Redisson 的 `RLock`，它内置看门狗续期，避免业务未执行完锁就过期。

---

## 参数校验 / Validation

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 常用注解速查

| 注解 | 说明 |
| --- | --- |
| `@NotNull` | 不能为 null |
| `@NotBlank` | 字符串非空且去除空格后非空 |
| `@NotEmpty` | 集合/数组非空 |
| `@Size(min, max)` | 长度范围 |
| `@Min` / `@Max` | 数值范围 |
| `@Email` | 邮箱格式 |
| `@Pattern(regexp)` | 正则匹配 |
| `@Positive` / `@PositiveOrZero` | 正数 / 非负数 |
| `@Valid` | 级联校验嵌套对象 |
| `@Validated` | 类级校验（配合分组/方法参数校验） |

```java
import jakarta.validation.constraints.*;

@Data
public class UserDTO {
    @NotNull(message = "用户 id 不能为空")
    private Long id;

    @NotBlank(message = "用户名不能为空")
    @Size(min = 2, max = 20, message = "用户名长度 2~20")
    private String username;

    @Email(message = "邮箱格式错误")
    private String email;

    @Min(value = 1, message = "年龄不能小于 1")
    @Max(value = 150, message = "年龄不能大于 150")
    private Integer age;

    @Valid                                   // 级联校验
    private AddressDTO address;
}
```

```java
@RestController
public class UserController {

    /** @RequestBody + @Valid 触发对象校验，失败抛 MethodArgumentNotValidException */
    @PostMapping("/users")
    public Result<Void> create(@RequestBody @Valid UserDTO dto) {
        return Result.ok(null);
    }

    /** 方法参数校验需在类上加 @Validated，失败抛 ConstraintViolationException */
    @Validated
    @GetMapping("/users/{id}")
    public Result<UserDTO> get(@PathVariable @Min(1) Long id) {
        return Result.ok(null);
    }
}
```

!!! note
    校验失败异常已在全局异常处理器统一转换，业务代码里不必再手写 if-else 判断。

---

## 接口文档 / API Docs（Knife4j · springdoc-openapi）

### 依赖

```xml
<!-- springdoc-openapi：Spring Boot 3 首选 -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>

<!-- Knife4j：增强 UI（基于 springdoc），提供 doc.html 聚合界面 -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.4.0</version>
</dependency>
```

### 配置

```yaml
springdoc:
  swagger-ui:
    path: /swagger-ui.html          # 原生 UI 访问路径
  api-docs:
    path: /v3/api-docs

knife4j:
  enable: true                      # 开启 doc.html 增强界面
  setting:
    language: zh_cn
```

### 常用注解

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.Parameter;

@Tag(name = "用户管理", description = "用户相关接口")
@RestController
@RequestMapping("/users")
public class UserController {

    @Operation(summary = "创建用户")
    @PostMapping
    public Result<Void> create(@RequestBody @Valid UserDTO dto) {
        return Result.ok(null);
    }

    @Operation(summary = "查询用户")
    @GetMapping("/{id}")
    public Result<UserDTO> get(
            @Parameter(description = "用户 id") @PathVariable Long id) {
        return Result.ok(null);
    }
}
```

!!! tip
    访问 `http://localhost:8080/doc.html` 查看 Knife4j 增强界面；生产环境记得通过配置关闭文档或加上鉴权。

---

## 定时任务 / Scheduled Tasks

### @Scheduled（单机）

```java
@Configuration
@EnableScheduling                          // 开启定时任务支持
public class ScheduleConfig {

    /** 每 5 秒执行一次（fixedRate：上次开始时间算起；fixedDelay：上次结束算起） */
    @Scheduled(fixedRate = 5000)
    public void report() {
        // ...
    }

    /** cron 表达式：每天凌晨 2 点执行 */
    @Scheduled(cron = "0 0 2 * * ?")
    public void dailyJob() {
        // ...
    }
}
```

```yaml
# 自定义线程池，避免只有一个调度线程互相阻塞
spring:
  task:
    scheduling:
      pool:
        size: 4
```

cron 表达式为 6 位（秒 分 时 日 月 周）：`0 0/5 * * * ?` 表示「每 5 分钟」。

!!! warning
    多实例部署时 `@Scheduled` 会在每个节点都执行，导致重复执行，需要用分布式锁或改用分布式调度框架。

### xxl-job 简介

分布式任务调度平台，包含调度中心（admin）与执行器（executor）。

```xml
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>2.4.1</version>
</dependency>
```

```yaml
xxl:
  job:
    admin:
      addresses: http://127.0.0.1:8088/xxl-job-admin
    executor:
      appname: demo-executor
      address: ""
      ip: ""
      port: 9999
      logpath: ./logs/xxl-job
      logretentiondays: 30
    accessToken: default_token
```

```java
@XxlJob("demoJobHandler")
public void demoJob() {
    XxlJobHelper.log("任务开始执行");
    // 业务逻辑
    XxlJobHelper.handleSuccess("执行完成");
}
```

| 对比 | @Scheduled | xxl-job |
| --- | --- | --- |
| 部署 | 单机 | 分布式集群 |
| 调度 | 本地 | 调度中心统一管理 |
| 失败重试 | 无 | 支持 |
| 可视化 | 无 | Web 控制台 |

---

## 注册中心与配置中心 / Nacos Registry & Config

### 依赖

```xml
<!-- 注册中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!-- 配置中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!-- Spring Cloud 2023 起，读取配置推荐引入 bootstrap 替代方案 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 配置

```yaml
spring:
  application:
    name: demo-service
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      username: nacos
      password: nacos
      discovery:
        namespace: public
        group: DEFAULT_GROUP
      config:
        namespace: public
        group: DEFAULT_GROUP
        file-extension: yaml
  config:
    import:
      - optional:nacos:demo-service.yaml     # Spring Boot 3 推荐用 import 方式
```

```java
@RestController
@RefreshScope                                // 配置变更后自动刷新
public class ConfigController {

    @Value("${demo.version:1.0}")
    private String version;

    @GetMapping("/version")
    public Result<String> version() {
        return Result.ok(version);
    }
}
```

!!! note
    Nacos 配置的 DataId 约定为 `${spring.application.name}-${profile}.${file-extension}`，改名时注意同步。

!!! tip
    启动类加 `@EnableDiscoveryClient`（新版可省略，有 starter 即自动启用），服务即可被其他服务通过服务名发现与负载均衡。

---

## 熔断限流 / Sentinel

### 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<!-- 如需在代码中定义规则 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
</dependency>
```

### 配置

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8858          # Sentinel 控制台地址
        port: 8719
      eager: true                          # 启动即初始化，无需首次请求
      web-context-unify: false             # 关闭 URL 归一化，便于区分监控项
```

### 代码定义规则

```java
@Component
public class SentinelRuleConfig {

    @PostConstruct
    public void init() {
        // 限流规则：QPS 上限 100
        FlowRule flow = new FlowRule();
        flow.setResource("orderQuery");
        flow.setGrade(RuleConstant.FLOW_GRADE_QPS);
        flow.setCount(100);
        FlowRuleManager.loadRules(Collections.singletonList(flow));

        // 熔断规则：异常比例 > 0.5 时熔断 10 秒
        DegradeRule degrade = new DegradeRule("orderQuery");
        degrade.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO);
        degrade.setCount(0.5);
        degrade.setTimeWindow(10);
        DegradeRuleManager.loadRules(Collections.singletonList(degrade));
    }
}
```

```java
// 兜底方法：@SentinelResource 触发限流/熔断时回调
@SentinelResource(value = "orderQuery", blockHandler = "queryBlocked")
public Order queryOrder(Long id) {
    return orderMapper.selectById(id);
}

public Order queryBlocked(Long id, BlockException ex) {
    throw new BizException(429, "请求过于频繁，请稍后重试");
}
```

| 概念 | 说明 |
| --- | --- |
| 资源（Resource） | 被保护的对象，接口或方法 |
| 限流（Flow） | 按 QPS/线程数控制流量 |
| 熔断降级（Degrade） | 异常比例/慢调用达到阈值时快速失败 |
| 热点参数 | 对特定参数值限流 |
| 系统规则 | 保护整体负载 |

---

## 分布式事务 / Seata

Seata 通过 TC（事务协调器）+ TM（事务管理器）+ RM（资源管理器）实现分布式事务。

### 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

### 配置

```yaml
seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: demo_tx_group
  service:
    vgroup-mapping:
      demo_tx_group: default
    grouplist:
      default: 127.0.0.1:8091
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: public
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: public
```

### 使用（AT 模式）

```java
@Service
public class OrderService {

    /** 全局事务入口，放在业务发起方 */
    @GlobalTransactional(rollbackFor = Exception.class)
    public void createOrder(OrderDTO dto) {
        orderMapper.insert(buildOrder(dto));   // 本地分支事务
        storageClient.deduct(dto.getSkuId(), dto.getCount());  // 远程分支事务
        accountClient.debit(dto.getUserId(), dto.getAmount());
    }
}
```

!!! warning
    AT 模式要求各参与库存在 `undo_log` 表；跨服务调用必须传递 `XID`（Feign / RestTemplate 需引入 Seata 拦截器）。

| 模式 | 特点 |
| --- | --- |
| AT | 自动补偿，无侵入，依赖 undo_log |
| TCC | 手动 Try/Confirm/Cancel，性能高 |
| SAGA | 长事务，状态机补偿 |
| XA | 强一致，性能较低 |

---

## 优雅停机与日志 / Graceful Shutdown & Logging

### 优雅停机

```yaml
server:
  shutdown: graceful                 # 开启优雅停机

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # 等待处理中的请求完成，超时后强制关闭
```

!!! note
    Kubernetes 环境下还需配合 `preStop` 钩子与 `terminationGracePeriodSeconds`，让流量先摘除再停机。

### logback-spring.xml 要点

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 引用 Spring Boot 默认配置，再覆盖 -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <!-- 控制台输出，带 traceId -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [%X{traceId}] %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 滚动文件：按天 + 按大小 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/demo-service.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/demo-service.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>5GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%X{traceId}] %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 异步包装，降低 I/O 阻塞 -->
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="FILE"/>
        <queueSize>1024</queueSize>
        <discardingThreshold>0</discardingThreshold>
    </appender>

    <!-- 日志级别：按包控制 -->
    <logger name="com.example.demo" level="DEBUG"/>
    <logger name="org.springframework" level="WARN"/>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_FILE"/>
    </root>

</configuration>
```

!!! tip
    `%X{traceId}` 对应前面 Filter 里 `MDC.put("traceId", ...)`，可在日志中串联同一请求的所有记录；分环境可用 `<springProfile name="prod">` 控制输出目标。

---

## 中间件 Docker 启动合集 / Docker Cheatsheet

```bash
# Redis（带密码）
docker run -d --name redis -p 6379:6379 \
  redis:7 redis-server --requirepass root123

# Nacos 单机模式（2.x 需额外映射 9848/9849 gRPC 端口）
docker run -d --name nacos -e MODE=standalone \
  -p 8848:8848 -p 9848:9848 -p 9849:9849 \
  nacos/nacos-server:v2.3.2

# Sentinel 控制台
docker run -d --name sentinel -p 8858:8858 \
  bladex/sentinel-dashboard:1.8.6

# Seata Server（需先准备配置文件 registry.conf / file.conf 挂载）
docker run -d --name seata-server -p 8091:8091 -p 7091:7091 \
  -e SEATA_IP=127.0.0.1 \
  seataio/seata-server:2.0.0

# MySQL 8
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root \
  -p 3306:3306 mysql:8.0 --default-authentication-plugin=mysql_native_password

# RocketMQ NameServer + Broker（简版，生产需配置存储与网络）
docker run -d --name rmqnamesrv -p 9876:9876 apache/rocketmq:5.1.0 sh mqnamesrv
```

| 中间件 | 默认端口 | 控制台/备注 |
| --- | --- | --- |
| Redis | 6379 | 无控制台，用 redis-cli |
| Nacos | 8848 (+9848/9849) | `http://ip:8848/nacos` |
| Sentinel | 8858 | `http://ip:8858` |
| Seata | 8091 (+7091) | 7091 为控制台端口 |
| MySQL | 3306 | — |
| RocketMQ | 9876 | 需另启 Broker |

!!! warning
    以上均为开发测试用「快速启动」命令，数据默认不持久化；生产部署务必挂载数据卷、配置持久化与鉴权。

!!! tip
    建议用 `docker compose` 编排上述中间件，一条 `docker compose up -d` 拉起整套本地环境，避免每次手敲长命令。
