---
title: MyBatis-Plus
tags:
  - 后端
  - Spring Boot
  - ORM
  - MyBatis-Plus
---

## 介绍 / Introduction

MyBatis-Plus（MP）是 MyBatis 的增强工具，只做增强不做改变：在不破坏原生 MyBatis 能力的前提下，提供 CRUD 封装、条件构造器、分页、代码生成等开箱即用功能。

!!! note
    引入 MP 后仍可继续使用原生 MyBatis 的 XML 映射，两者可共存。

## 核心概念与优势 / Core Concept

| 维度 | 原生 MyBatis | MyBatis-Plus |
| --- | --- | --- |
| 单表 CRUD | 每个方法都要写 XML/SQL | `BaseMapper` 内置，零 SQL |
| 条件拼接 | 手写 `<if test>` 动态 SQL | `Wrapper` 链式构造 |
| 分页 | 手写 `limit` / 分页插件自配 | 内置 `PaginationInnerInterceptor` |
| 逻辑删除 | 手写 `where deleted=0` | 注解/全局配置自动生效 |
| 自动填充 | 手动 set 时间字段 | `MetaObjectHandler` 自动填充 |
| 代码生成 | 手写或用 MBG | `FastAutoGenerator` 一键生成 |
| 扩展能力 | 无 | 乐观锁、多租户、防全表更新等 |

!!! tip
    MP 的主战场是「单表增删改查」。复杂多表关联统计，仍推荐写 XML 或自定义 SQL。

## 依赖与配置 / Dependencies & Config

### Maven 依赖

```xml
<!-- Spring Boot 3.x 使用 spring-boot3-starter -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.7</version>
</dependency>

<!-- MySQL 驱动 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

### application.yml

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: root

mybatis-plus:
  # Mapper XML 位置（纯注解方式可省略）
  mapper-locations: classpath*:/mapper/**/*.xml
  # 实体包别名，多模块/多包场景需显式指定
  type-aliases-package: com.example.demo.entity
  configuration:
    # 开启下划线转驼峰（默认即 true）
    map-underscore-to-camel-case: true
    # 打印 SQL，仅开发环境使用
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    banner: false
    db-config:
      # 全局主键策略（默认 ASSIGN_ID 雪花算法）
      id-type: auto
      # 逻辑删除字段名
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
```

!!! warning
    `log-impl: StdOutImpl` 会把每条 SQL 打到控制台，性能开销较大，**生产环境务必关闭**。

### 启动类扫描

```java
@SpringBootApplication
@MapperScan("com.example.demo.mapper") // 一次性扫描，免去逐个 @Mapper
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

## 实体注解 / Entity Annotations

```java
@Data
@TableName("sys_user")                       // 表名映射（表名与类名不一致时）
public class User {

    @TableId(type = IdType.AUTO)             // 主键策略：数据库自增
    private Long id;

    @TableField("user_name")                 // 字段名映射
    private String name;

    @TableField(exist = false)               // 非数据库字段
    private String token;

    @TableField(select = false)              // 查询时不返回该列（如密码）
    private String password;

    @TableLogic                              // 逻辑删除标记字段
    private Integer deleted;

    @TableField(fill = FieldFill.INSERT)     // 插入时自动填充
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE) // 插入和更新都填充
    private LocalDateTime updateTime;

    @Version                                 // 乐观锁版本号
    private Integer version;
}
```

### 主键策略 IdType

| 取值 | 说明 |
| --- | --- |
| `AUTO` | 数据库自增，需表设置 `AUTO_INCREMENT` |
| `NONE` | 不设置，跟随全局 `id-type`（默认 `ASSIGN_ID`） |
| `INPUT` | 手动赋值，插入前自行 set |
| `ASSIGN_ID` | 雪花算法生成 Long/String（默认） |
| `ASSIGN_UUID` | 生成 UUID 字符串 |

## BaseMapper 内置 CRUD / Built-in CRUD

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {}
```

```java
// 增
userMapper.insert(user);

// 删
userMapper.deleteById(1L);
userMapper.deleteBatchIds(Arrays.asList(1L, 2L, 3L));
userMapper.delete(Wrappers.<User>lambdaQuery().eq(User::getAge, 18));

// 改
User u = new User();
u.setId(1L);
u.setName("张三");
userMapper.updateById(u);                       // 按主键更新非空字段
userMapper.update(u, Wrappers.<User>lambdaQuery().eq(User::getAge, 18));

// 查
User one = userMapper.selectById(1L);
List<User> list = userMapper.selectBatchIds(Arrays.asList(1L, 2L));
List<User> all = userMapper.selectList(null);   // 传 null 表示无条件
User first = userMapper.selectOne(Wrappers.<User>lambdaQuery().eq(User::getName, "张三"));
Long count = userMapper.selectCount(null);
```

!!! tip
    `updateById` 只更新实体中**非 null** 的字段；要把某字段置空，用 `UpdateWrapper` 显式 `set`。

## 条件构造器 / Wrapper

### QueryWrapper（字符串列名）

```java
QueryWrapper<User> qw = new QueryWrapper<>();
qw.select("id", "user_name", "age")             // 指定返回列
  .eq("age", 18)
  .like("user_name", "张")
  .orderByDesc("create_time");

List<User> users = userMapper.selectList(qw);
```

### LambdaQueryWrapper（推荐，编译期安全）

```java
LambdaQueryWrapper<User> lqw = Wrappers.lambdaQuery();
lqw.eq(User::getAge, 18)
   .ne(User::getName, "李四")
   .gt(User::getAge, 10)
   .like(User::getName, "张")
   .in(User::getId, Arrays.asList(1L, 2L, 3L))
   .between(User::getAge, 18, 30)
   .orderByAsc(User::getAge)
   .orderByDesc(User::getCreateTime);

List<User> users = userMapper.selectList(lqw);
```

### UpdateWrapper

```java
UpdateWrapper<User> uw = new UpdateWrapper<>();
uw.set("age", 20)                        // set 字段
  .set("user_name", null)                // 允许置空
  .eq("id", 1L);
userMapper.update(null, uw);             // 实体传 null，仅按 wrapper 更新
```

### 常用方法速查

| 方法 | 作用 |
| --- | --- |
| `eq` / `ne` | 等于 / 不等于 |
| `gt` / `ge` | 大于 / 大于等于 |
| `lt` / `le` | 小于 / 小于等于 |
| `like` / `notLike` | 模糊包含 / 不包含 |
| `likeLeft` / `likeRight` | `%值` / `值%` |
| `in` / `notIn` | 包含 / 排除集合 |
| `between` / `notBetween` | 区间查询 |
| `isNull` / `isNotNull` | 空 / 非空 |
| `groupBy` / `having` | 分组 / 分组过滤 |
| `orderByAsc` / `orderByDesc` | 升序 / 降序 |
| `and` / `or` | 连接条件（注意加括号包裹） |
| `select` | 指定查询列 |

!!! warning
    使用 `or()` 时务必用括号包裹子条件，否则会与前后条件直接拼接造成语义错误：
    `qw.eq(...).and(w -> w.eq(...).or().eq(...))`。

## 分页插件 / Pagination

### 配置拦截器

```java
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 分页插件，指定数据库类型
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        // 乐观锁插件
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

### 使用

```java
Page<User> page = new Page<>(1, 10);              // 当前页, 每页条数
// page.setSearchCount(false);                    // 不需要 total 时可关闭 count
Page<User> result = userMapper.selectPage(page, Wrappers.<User>lambdaQuery().ge(User::getAge, 18));

result.getRecords();     // 当前页数据 List
result.getTotal();       // 总条数
result.getPages();       // 总页数
```

!!! note
    `PaginationInnerInterceptor` 需放在其他拦截器之前添加，否则分页可能不生效。

## 自动填充 / Auto Fill

```java
@Data
public class User {
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        // 严格填充：字段必须有 @TableField(fill=...) 才生效
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

## 乐观锁 / Optimistic Locking

```java
@Data
public class User {
    @Version
    private Integer version;   // 版本号字段，每次更新 +1
}
```

```java
// 配置见上文：interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor())

User u = userMapper.selectById(1L);
u.setAge(20);
int rows = userMapper.updateById(u);   // SQL: update ... set version=version+1 where id=? and version=?
// rows == 0 表示版本冲突，更新失败，需重试或提示
```

!!! warning
    乐观锁仅在 `updateById` 与 `update(entity, wrapper)` 中生效，`UpdateWrapper.set` 手动赋值不受保护。

## 逻辑删除 / Logic Delete

方式一：全局配置（见 `application.yml` 的 `logic-delete-field`）。

方式二：实体字段加注解：

```java
@TableLogic(value = "0", delval = "1")
private Integer deleted;
```

生效后：

```java
// 删除 → UPDATE user SET deleted=1 WHERE id=? AND deleted=0
userMapper.deleteById(1L);

// 查询自动追加 WHERE deleted=0
userMapper.selectList(null);
```

## 代码生成器 / Code Generator

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.7</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>
```

```java
public class CodeGen {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/demo?serverTimezone=Asia/Shanghai";
        FastAutoGenerator.create(url, "root", "root")
            .globalConfig(b -> b
                .author("磊哥")
                .outputDir(System.getProperty("user.dir") + "/src/main/java")
                .disableOpenDir()
                .commentDate("yyyy-MM-dd"))
            .packageConfig(b -> b
                .parent("com.example.demo")
                .entity("entity").mapper("mapper")
                .service("service").serviceImpl("service.impl")
                .controller("controller"))
            .strategyConfig(b -> b
                .addInclude("sys_user")          // 指定生成的表
                .addTablePrefix("sys_")          // 去除表前缀
                .entityBuilder().enableLombok().logicDeleteColumnName("deleted")
                .controllerBuilder().enableRestStyle())
            .execute();
    }
}
```

## 多数据源 / Dynamic DataSource

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot3-starter</artifactId>
    <version>4.3.1</version>
</dependency>
```

```yaml
spring:
  datasource:
    dynamic:
      primary: master          # 默认数据源
      strict: false            # 未匹配到数据源时是否报错
      datasource:
        master:
          url: jdbc:mysql://localhost:3306/db_master?serverTimezone=Asia/Shanghai
          username: root
          password: root
        slave:
          url: jdbc:mysql://localhost:3306/db_slave?serverTimezone=Asia/Shanghai
          username: root
          password: root
```

```java
@Service
public class UserService {
    @DS("slave")                 // 该方法走从库
    public List<User> listFromSlave() {
        return userMapper.selectList(null);
    }
}
```

!!! tip
    `@DS` 可作用于方法或类；未标注时按 `primary` 数据源执行。事务方法中也支持通过 `@DS` 切换。

## 常见坑与最佳实践 / Pitfalls & Best Practices

| 问题 | 现象 | 解决 |
| --- | --- | --- |
| Lambda 缓存告警 | `can not find lambda cache for this entity` | 用 `Wrappers.<User>lambdaQuery()` 显式类型，避免 `selectOne(wrapper)` 匿名泛型 |
| 批量插入慢 | `saveBatch` 逐条提交 | 加 `rewriteBatchedStatements=true`，或配置 `insertBatchSql` |
| 更新置空失败 | `updateById` 不更新 null 字段 | 用 `UpdateWrapper.set` 显式赋值 |
| 分页不生效 | 返回全量数据 | 确认已加 `PaginationInnerInterceptor` |
| 字段未填充 | 时间字段为 null | 检查处理器 `strict*` 方法与 `@TableField(fill)` 是否配套 |
| 条件语义错乱 | `or` 拼接异常 | 用 `and(w -> ...)` / `or(w -> ...)` 括号包裹 |
| SQL 无法注入 | 未扫到 Mapper | 启动类加 `@MapperScan` |

```java
// 批量插入优化：实体主键策略设为 INPUT 并使用 saveBatch
// 或直接重写 insertBatchSomeColumn 自定义批量方法
boolean ok = userService.saveBatch(userList, 500);  // 每 500 条一批提交
```

!!! tip
    - 单表用 `BaseMapper`，多表关联写 XML 或 `@Select` 注解。
    - 生产环境禁止开启 `StdOutImpl`，日志交给 `logging.level` 控制。
    - 复杂查询优先 `LambdaQueryWrapper`，列名重构时编译期即可发现问题。
