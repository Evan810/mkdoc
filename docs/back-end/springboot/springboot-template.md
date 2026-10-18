---
title: Spring Boot 项目模板
tags:
  - 后端
  - Spring Boot
  - 脚手架
  - 最佳实践
---

## 介绍 / Introduction

整理一套可直接 fork 使用的 Spring Boot 项目模板，覆盖分层结构、统一返回、全局异常、参数校验、拦截器、JWT 鉴权、多环境配置与容器化部署。

!!! note
    模板基于 Spring Boot 3.x + MyBatis-Plus + Knife4j，个人仓库地址见文末链接，欢迎 Star / Fork。

## 推荐项目分层结构 / Project Structure

```bash
springboot-template/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
└── src
    ├── main
    │   ├── java/com/example/template
    │   │   ├── TemplateApplication.java      # 启动类
    │   │   ├── common                        # 通用模块
    │   │   │   ├── result/R.java             # 统一返回
    │   │   │   ├── result/ResultCode.java    # 状态码枚举
    │   │   │   ├── exception/BusinessException.java
    │   │   │   ├── exception/GlobalExceptionHandler.java
    │   │   │   └── utils/                    # 工具类
    │   │   ├── config                        # 配置类
    │   │   │   ├── MybatisPlusConfig.java
    │   │   │   ├── SecurityConfig.java
    │   │   │   ├── SwaggerConfig.java
    │   │   │   └── WebMvcConfig.java
    │   │   ├── interceptor                   # 拦截器
    │   │   ├── controller                    # 接口层
    │   │   ├── service                       # 业务层（+ impl）
    │   │   ├── mapper                        # DAO 层
    │   │   ├── entity                        # 数据库实体
    │   │   ├── dto                           # 入参对象
    │   │   └── vo                            # 出参对象
    │   └── resources
    │       ├── application.yml               # 主配置
    │       ├── application-dev.yml           # 开发环境
    │       ├── application-prod.yml          # 生产环境
    │       ├── logback-spring.xml            # 日志配置
    │       └── mapper/                       # MyBatis XML
    └── test
        └── java/com/example/template
            └── TemplateApplicationTests.java # 测试入口
```

!!! tip
    分层职责：`controller` 只做参数接收与结果返回，业务逻辑全部下沉到 `service`，`mapper` 只负责数据访问。

## 统一返回结果 / Unified Result

```java
@Data
public class R<T> implements Serializable {
    private int code;
    private String msg;
    private T data;

    public static <T> R<T> ok() { return build(ResultCode.SUCCESS, null); }
    public static <T> R<T> ok(T data) { return build(ResultCode.SUCCESS, data); }
    public static <T> R<T> fail(ResultCode rc) { return build(rc, null); }

    public static <T> R<T> fail(int code, String msg) {
        R<T> r = new R<>();
        r.setCode(code);
        r.setMsg(msg);
        return r;
    }

    private static <T> R<T> build(ResultCode rc, T data) {
        R<T> r = new R<>();
        r.setCode(rc.getCode());
        r.setMsg(rc.getMsg());
        r.setData(data);
        return r;
    }
}
```

```java
@Getter
@AllArgsConstructor
public enum ResultCode {
    SUCCESS(200, "操作成功"),
    FAIL(500, "操作失败"),
    PARAM_ERROR(400, "参数校验失败"),
    UNAUTHORIZED(401, "未登录或 Token 失效"),
    FORBIDDEN(403, "无访问权限"),
    NOT_FOUND(404, "资源不存在");

    private final int code;
    private final String msg;
}
```

## 全局异常处理 / Global Exception Handler

```java
@Getter
public class BusinessException extends RuntimeException {
    private final int code;

    public BusinessException(ResultCode rc) {
        super(rc.getMsg());
        this.code = rc.getCode();
    }

    public BusinessException(String msg) {
        super(msg);
        this.code = ResultCode.FAIL.getCode();
    }
}
```

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 业务异常：可预期，返回具体提示
    @ExceptionHandler(BusinessException.class)
    public R<Void> handleBusiness(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return R.fail(e.getCode(), e.getMessage());
    }

    // 参数校验异常
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public R<Void> handleValid(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldError().getDefaultMessage();
        return R.fail(ResultCode.PARAM_ERROR.getCode(), msg);
    }

    // 兜底异常：记录堆栈，不向外暴露细节
    @ExceptionHandler(Exception.class)
    public R<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return R.fail(ResultCode.FAIL);
    }
}
```

!!! warning
    兜底异常务必打印完整堆栈便于排查，但返回给前端的信息应保持模糊，避免泄露内部实现。

## 通用工具类 / Common Utils

```java
public class DateUtils {
    public static final String PATTERN = "yyyy-MM-dd HH:mm:ss";

    public static String format(LocalDateTime time) {
        return time == null ? null
            : time.format(DateTimeFormatter.ofPattern(PATTERN));
    }

    public static LocalDateTime parse(String text) {
        return LocalDateTime.parse(text, DateTimeFormatter.ofPattern(PATTERN));
    }
}
```

```java
public class BeanUtils {
    /** 浅拷贝属性（同名同类型） */
    public static <T> T copy(Object source, Class<T> targetClass) {
        T target = BeanUtils.instantiateClass(targetClass);
        org.springframework.beans.BeanUtils.copyProperties(source, target);
        return target;
    }

    /** 列表批量转换 */
    public static <S, T> List<T> copyList(List<S> list, Class<T> clazz) {
        return list.stream().map(s -> copy(s, clazz)).collect(Collectors.toList());
    }
}
```

!!! note
    字符串判空优先使用 `org.springframework.util.StringUtils.hasText` 或 Hutool 的 `StrUtil.isBlank`，避免手写 `== null || ""`。

## 参数校验与分组校验 / Validation & Groups

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
public interface ValidGroups {
    interface Create {}
    interface Update {}
}

@Data
public class UserDTO {
    @NotNull(message = "id 不能为空", groups = ValidGroups.Update.class)
    private Long id;

    @NotBlank(message = "用户名不能为空", groups = {ValidGroups.Create.class, ValidGroups.Update.class})
    private String username;

    @Min(value = 1, message = "年龄不能小于 1")
    private Integer age;
}
```

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @PostMapping
    public R<Void> create(@RequestBody @Validated(ValidGroups.Create.class) UserDTO dto) {
        // 校验通过才进入业务
        return R.ok();
    }

    @PutMapping
    public R<Void> update(@RequestBody @Validated(ValidGroups.Update.class) UserDTO dto) {
        return R.ok();
    }
}
```

| 注解 | 作用 |
| --- | --- |
| `@NotNull` | 不能为 null |
| `@NotBlank` | 字符串非空且非空白 |
| `@NotEmpty` | 集合/字符串非空 |
| `@Size` / `@Length` | 长度区间 |
| `@Min` / `@Max` | 数值边界 |
| `@Pattern` | 正则匹配 |
| `@Email` | 邮箱格式 |

## 拦截器实现 / Interceptors

```java
@Slf4j
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler) {
        if (!(handler instanceof HandlerMethod)) {
            return true;                                  // 静态资源等直接放行
        }
        long start = System.currentTimeMillis();
        String token = req.getHeader("Authorization");
        log.info("请求 {} {} 开始", req.getMethod(), req.getRequestURI());

        if (StrUtil.isBlank(token)) {
            throw new BusinessException(ResultCode.UNAUTHORIZED);
        }
        // 实际校验交给 JwtUtil（见下文）
        req.setAttribute("start", start);
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse resp,
                                Object handler, Exception ex) {
        Long start = (Long) req.getAttribute("start");
        if (start != null) {
            log.info("请求 {} 耗时 {} ms", req.getRequestURI(),
                System.currentTimeMillis() - start);
        }
    }
}
```

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Resource
    private AuthInterceptor authInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/auth/login", "/doc.html",
                        "/webjars/**", "/swagger-resources/**", "/v3/api-docs/**");
    }
}
```

## JWT + Spring Security 鉴权 / Auth

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
```

### JWT 工具

```java
public class JwtUtil {
    private static final String SECRET = "change-me-secret-key-at-least-32-bytes!!";
    private static final long EXPIRE = 7 * 24 * 3600_000L;   // 7 天

    public static String create(Long userId, String username) {
        return Jwts.builder()
            .subject(username)
            .claim("userId", userId)
            .expiration(new Date(System.currentTimeMillis() + EXPIRE))
            .signWith(Keys.hmacShaKeyFor(SECRET.getBytes()), Jwts.SIG.HS256)
            .compact();
    }

    public static Claims parse(String token) {
        return Jwts.parser()
            .verifyWith(Keys.hmacShaKeyFor(SECRET.getBytes()))
            .build()
            .parseSignedClaims(token)
            .getPayload();
    }
}
```

### Security 配置

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private static final String[] WHITE_LIST = {
        "/auth/login", "/doc.html", "/webjars/**", "/v3/api-docs/**"
    };

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(WHITE_LIST).permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 鉴权流程

```bash
1. 客户端 POST /auth/login 提交用户名 + 密码
2. 服务端校验密码（BCrypt），通过后 JwtUtil.create() 签发 token
3. 客户端后续请求在 Header 带 Authorization: Bearer <token>
4. JwtAuthenticationFilter 解析 token 并写入 SecurityContext
5. 解析失败 / 过期 → 返回 401
```

!!! tip
    `JwtAuthenticationFilter` 只需从 Header 取出 token，`JwtUtil.parse` 成功则构造 `UsernamePasswordAuthenticationToken` 放入上下文，失败直接放行交由 Security 拦截。

## Swagger / Knife4j / API Docs

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.4.0</version>
</dependency>
```

```yaml
springdoc:
  swagger-ui:
    path: /swagger-ui.html
  api-docs:
    path: /v3/api-docs
knife4j:
  enable: true
  setting:
    language: zh_cn
```

```java
@Bean
public OpenAPI openApi() {
    return new OpenAPI()
        .info(new Info().title("项目模板 API").version("v1.0")
            .description("Spring Boot 项目模板接口文档"));
}
```

!!! note
    访问 `http://localhost:8080/doc.html` 打开 Knife4j 增强界面；生产环境建议关闭文档或加权限保护。

## 多环境配置 / Multi-Environment

```yaml
# application.yml —— 只保留公共配置与 profile 开关
spring:
  profiles:
    active: dev
```

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo?serverTimezone=Asia/Shanghai
    username: root
    password: root
logging:
  level:
    com.example.template: debug
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/demo?serverTimezone=Asia/Shanghai
    username: ${DB_USER}
    password: ${DB_PWD}
logging:
  level:
    root: warn
```

!!! warning
    生产环境敏感信息（数据库密码、密钥）应通过环境变量或配置中心注入，不要硬编码并提交到仓库。

## 日志配置要点 / Logging

```yaml
logging:
  level:
    root: info
    com.example.template: debug
  file:
    name: logs/app.log
  logback:
    rollingpolicy:
      max-file-size: 50MB
      max-history: 15
```

| 要点 | 说明 |
| --- | --- |
| 分级输出 | 生产 `warn`，开发 `debug`，避免全量打印 |
| 文件滚动 | 按大小/日期切分，限制保留天数 |
| 异步日志 | 高并发用 `AsyncAppender` 降低 I/O 阻塞 |
| 敏感信息 | 禁止打印密码、Token、身份证等 |
| 链路追踪 | 建议日志带 `traceId`，便于问题定位 |

## Dockerfile 与 docker-compose

```dockerfile
# 多阶段构建：先构建再加运行
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn -B dependency:go-offline
COPY src ./src
RUN mvn -B clean package -DskipTests

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "-Duser.timezone=Asia/Shanghai", "app.jar"]
```

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_USER=root
      - DB_PWD=root
    depends_on:
      - mysql
    restart: always
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: demo
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
volumes:
  mysql-data:
```

## 打包部署 / Build & Deploy

```bash
# 打包（跳过测试）
mvn clean package -DskipTests

# 本地运行
java -jar target/template-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev

# 后台启动（生产）
nohup java -jar app.jar --spring.profiles.active=prod > app.log 2>&1 &

# 查看日志 / 进程
tail -f app.log
ps -ef | grep app.jar

# 停止服务
kill -9 <pid>

# Docker 构建与运行
docker build -t springboot-template:1.0 .
docker compose up -d
```

| 命令 | 用途 |
| --- | --- |
| `mvn clean package` | 打包生成可执行 jar |
| `-DskipTests` | 跳过测试加速构建 |
| `--spring.profiles.active` | 指定生效环境 |
| `nohup ... &` | 后台常驻运行 |
| `docker compose up -d` | 容器编排一键启动 |

!!! tip
    Jenkins / GitHub Actions 等 CI 场景，直接复用 `mvn clean package -DskipTests` 与 `Dockerfile` 两步即可完成自动化构建与镜像推送。
