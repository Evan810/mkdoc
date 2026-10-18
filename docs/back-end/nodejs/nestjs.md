---
title: NestJS
tags:
  - 后端
  - Node.js
  - NestJS
  - TypeScript
  - 依赖注入
---

## 介绍 / Introduction

NestJS 是基于 TypeScript 的企业级 Node.js 框架，借鉴 Angular 风格的模块化、装饰器与依赖注入（DI）体系，
开箱即用地提供管道、守卫、拦截器、异常过滤器等能力，适合构建可维护的大型服务端应用。

!!! note
    NestJS 底层默认使用 Express，可通过 `NestFactory.create(AppModule, { adapter: new FastifyAdapter() })` 切换为 Fastify 以获得更高性能。

## 快速创建与 CLI 常用命令

```bash
# 全局安装 CLI
npm i -g @nestjs/cli

# 创建项目（含包管理器选择）
nest new my-nest-app
cd my-nest-app
npm run start:dev        # 开发模式（热重载）
```

| 命令 | 说明 |
| --- | --- |
| `nest new <name>` | 新建项目 |
| `nest g module users` | 生成模块 |
| `nest g controller users` | 生成控制器 |
| `nest g service users` | 生成服务 |
| `nest g resource users` | 一键生成 CRUD 资源（含 DTO） |
| `nest g guard auth` | 生成守卫 |
| `nest g interceptor transform` | 生成拦截器 |
| `nest g pipe validation` | 生成管道 |
| `nest g filter http-exception` | 生成异常过滤器 |
| `nest g middleware logger` | 生成中间件 |
| `npm run start:dev` | 开发热重载 |
| `npm run start:prod` | 生产模式 |
| `npm run build` | 编译到 `dist/` |
| `npm run test` / `test:e2e` | 单元测试 / 端到端测试 |

## 核心概念 / Core Concepts

Nest 应用由模块组成，模块聚合控制器与服务，通过依赖注入串联。

| 概念 | 装饰器 / 类 | 职责 |
| --- | --- | --- |
| Module | `@Module()` | 组织功能单元，声明依赖与导出 |
| Controller | `@Controller()` | 处理 HTTP 请求，路由分发 |
| Service（Provider） | `@Injectable()` | 承载业务逻辑，可被注入 |
| DI 容器 | 构造函数注入 | 自动装配 Provider 与依赖 |

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  imports: [],                     // 依赖的其他模块
  controllers: [UsersController],  // 本模块控制器
  providers: [UsersService],       // 本模块 Provider
  exports: [UsersService],         // 供其他模块使用
})
export class UsersModule {}
```

```typescript
// app.module.ts（根模块）
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';

@Module({
  imports: [UsersModule],
})
export class AppModule {}
```

!!! tip
    只有被某模块 `exports` 导出的 Provider，才能被导入该模块的其他模块注入使用。

## 控制器装饰器 / Controller Decorators

```typescript
import {
  Controller, Get, Post, Put, Delete, Patch,
  Param, Query, Body, Headers, HttpCode, HttpStatus,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')               // 路由前缀 /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}  // 注入 Service

  @Get()                          // GET /users
  findAll(@Query('page') page = '1') {
    return this.usersService.findAll(Number(page));
  }

  @Get(':id')                     // GET /users/:id
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(Number(id));
  }

  @Post()                         // POST /users
  @HttpCode(HttpStatus.CREATED)   // 默认 POST 为 201，可显式指定
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Put(':id')                     // PUT /users/:id
  update(@Param('id') id: string, @Body() dto: CreateUserDto) {
    return this.usersService.update(Number(id), dto);
  }

  @Delete(':id')                  // DELETE /users/:id
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string) {
    return this.usersService.remove(Number(id));
  }
}
```

| 装饰器 | 作用 |
| --- | --- |
| `@Controller('users')` | 声明控制器与路由前缀 |
| `@Get / @Post / @Put / @Delete / @Patch` | 绑定 HTTP 方法 |
| `@Param('id')` | 取路径参数 |
| `@Query('page')` | 取查询参数 |
| `@Body()` | 取请求体 |
| `@Headers('x-token')` | 取请求头 |
| `@HttpCode(...)` | 指定响应状态码 |

## Provider 与依赖注入 / Providers & DI

```typescript
// users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';

@Injectable()                     // 标记为可注入 Provider
export class UsersService {
  private users = [{ id: 1, name: 'Alice' }];

  findAll(page: number) {
    return this.users;
  }

  findOne(id: number) {
    const user = this.users.find((u) => u.id === id);
    if (!user) throw new NotFoundException('用户不存在');
    return user;
  }
}
```

### 自定义 Provider（注入 token）

```typescript
// 1. 值提供者
@Module({
  providers: [{ provide: 'CONFIG', useValue: { key: 'value' } }],
})
export class AppModule {}

// 2. 工厂提供者（可注入其他依赖）
@Module({
  providers: [
    {
      provide: 'DB_POOL',
      useFactory: async (config: ConfigService) => createPool(config),
      inject: [ConfigService],
    },
  ],
})
export class AppModule {}

// 使用
constructor(@Inject('CONFIG') private config: { key: string }) {}
```

!!! note
    构造函数注入为默认方式；也可用属性注入 `@Inject()` 或可选注入 `@Optional()`。

## DTO 与参数校验 / DTO & Validation

```bash
npm install class-validator class-transformer
```

```typescript
// dto/create-user.dto.ts
import { IsString, IsInt, Min, Max, IsOptional, IsEmail } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsInt()
  @Min(0)
  @Max(150)
  age: number;

  @IsEmail()
  @IsOptional()
  email?: string;
}
```

```typescript
// main.ts 全局启用校验管道
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,            // 剥离 DTO 未声明的属性
      forbidNonWhitelisted: true, // 出现多余属性直接报错
      transform: true,            // 自动类型转换（如 string -> number）
    }),
  );
  await app.listen(3000);
}
bootstrap();
```

| 选项 | 说明 |
| --- | --- |
| `whitelist` | 过滤掉未在 DTO 中声明的字段 |
| `forbidNonWhitelisted` | 出现未知字段时报 400 |
| `transform` | 依据 DTO 类型自动转换 |
| `transformOptions` | 传给 class-transformer 的选项 |

## 管道 · 守卫 · 拦截器 · 异常过滤器 / Pipes · Guards · Interceptors · Filters

Nest 请求生命周期：中间件 → 守卫 → 拦截器(前) → 管道 → 控制器 → 拦截器(后) → 异常过滤器。

### 管道（Pipe）

```typescript
// 内置解析管道：将参数转为数字
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.usersService.findOne(id);
}
```

### 守卫（Guard，JWT 鉴权）

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwt: JwtService) {}

  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest();
    const token = (req.headers.authorization || '').replace('Bearer ', '');
    try {
      req.user = this.jwt.verify(token);   // 校验通过并挂载用户
      return true;
    } catch {
      return false;                        // 返回 false -> 403
    }
  }
}

// 使用：@UseGuards(JwtAuthGuard)
@UseGuards(JwtAuthGuard)
@Get('profile')
getProfile() { /* ... */ }
```

### 拦截器（Interceptor，统一响应）

```typescript
import {
  Injectable, NestInterceptor, ExecutionContext, CallHandler,
} from '@nestjs/common';
import { map } from 'rxjs/operators';

export interface ApiResponse<T> { code: number; message: string; data: T; }

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, ApiResponse<T>> {
  intercept(ctx: ExecutionContext, next: CallHandler) {
    return next.handle().pipe(
      map((data) => ({ code: 0, message: 'success', data })),
    );
  }
}

// 全局注册
app.useGlobalInterceptors(new TransformInterceptor());
```

### 异常过滤器（ExceptionFilter）

```typescript
import {
  ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus,
} from '@nestjs/common';

@Catch()                               // 捕获所有异常
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const res = ctx.getResponse();
    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    res.status(status).json({
      code: status,
      message:
        exception instanceof HttpException
          ? exception.message
          : '服务器内部错误',
      data: null,
    });
  }
}

// 全局注册
app.useGlobalFilters(new AllExceptionsFilter());
```

!!! tip
    内置异常如 `NotFoundException`、`BadRequestException`、`UnauthorizedException` 会被过滤器自动转成标准 JSON 响应。

## 中间件与配置 / Middleware & Config

```bash
npm install @nestjs/config
```

```typescript
// 函数式中间件
import { Injectable, NestMiddleware } from '@nestjs/common';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: any, res: any, next: () => void) {
    console.log(`${req.method} ${req.originalUrl}`);
    next();
  }
}

// 在模块中注册（需实现 NestModule）
@Module({ /* ... */ })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('users');   // 只对 users 生效
  }
}
```

```typescript
// 配置模块
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true, envFilePath: '.env' })],
})
export class AppModule {}
```

```bash
# .env
PORT=3000
JWT_SECRET=change-me
DB_HOST=127.0.0.1
```

```typescript
// 读取配置
constructor(private config: ConfigService) {
  const port = this.config.get<number>('PORT');
}
```

## 数据库集成 / Database

### TypeORM（实体 + Repository）

```bash
npm install @nestjs/typeorm typeorm mysql2
```

```typescript
// 注册模块
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: '127.0.0.1',
      port: 3306,
      username: 'root',
      password: 'secret',
      database: 'demo',
      entities: [User],
      synchronize: false,        // 生产环境务必关闭
    }),
    TypeOrmModule.forFeature([User]),
  ],
})
export class AppModule {}
```

```typescript
// user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ type: 'int', nullable: true })
  age: number;
}

// 在 Service 中注入 Repository
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User) private repo: Repository<User>,
  ) {}

  findAll() { return this.repo.find(); }
  findOne(id: number) { return this.repo.findOneBy({ id }); }
  create(data: Partial<User>) { return this.repo.save(data); }
}
```

### Prisma（简介）

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// schema.prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  age   Int?
}
```

```bash
npx prisma migrate dev --name init    # 迁移并生成客户端
```

```typescript
// 封装为 Service 使用
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient {}
```

| 方案 | 类型 | 特点 | 适用场景 |
| --- | --- | --- | --- |
| TypeORM | ORM | 装饰器实体、Repository 模式，与 Nest 深度集成 | 关系型、偏好装饰器风格 |
| Prisma | ORM | 强类型客户端、Schema 优先、迁移工具完善 | 类型安全、开发体验优先 |

## Redis 缓存 / Redis

```bash
npm install ioredis
```

```typescript
// cache.service.ts
import { Injectable, OnModuleDestroy } from '@nestjs/common';
import Redis from 'ioredis';

@Injectable()
export class CacheService implements OnModuleDestroy {
  private client = new Redis({ host: '127.0.0.1', port: 6379 });

  async get(key: string) { return this.client.get(key); }
  async set(key: string, value: string, ttl = 60) {
    await this.client.set(key, value, 'EX', ttl);
  }
  async del(key: string) { await this.client.del(key); }

  onModuleDestroy() { this.client.disconnect(); }
}
```

```typescript
// 使用：缓存优先
async findOne(id: number) {
  const cached = await this.cache.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  const user = await this.repo.findOneBy({ id });
  await this.cache.set(`user:${id}`, JSON.stringify(user), 300);
  return user;
}
```

## Swagger 文档 / Swagger

```bash
npm install @nestjs/swagger swagger-ui-express
```

```typescript
// main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('Demo API')
  .setDescription('接口文档')
  .setVersion('1.0')
  .addBearerAuth()               // 支持 Bearer JWT
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api-docs', app, document);   // 访问 /api-docs
```

```typescript
// 用装饰器补充文档信息
import { ApiTags, ApiOperation, ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ description: '用户名', example: 'Alice' })
  name: string;
}

@ApiTags('users')                // 分组标签
@Controller('users')
export class UsersController {
  @ApiOperation({ summary: '查询用户列表' })
  @Get()
  findAll() { /* ... */ }
}
```

!!! tip
    CLI 提供 Swagger 插件可自动从 DTO 与类型注解生成文档，减少手写 `@ApiProperty` 的负担。

## 微服务与任务调度简介 / Microservices & Scheduling

### 微服务

```bash
npm install @nestjs/microservices
```

```typescript
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';

const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.TCP,
  options: { host: '127.0.0.1', port: 3001 },
});
await app.listen();
```

```typescript
// 消息处理：@MessagePattern('user.created')
@MessagePattern('user.created')
handleUserCreated(data: { id: number }) { /* ... */ }
```

### 任务调度

```bash
npm install @nestjs/schedule
```

```typescript
import { ScheduleModule, Cron, Interval, Timeout } from '@nestjs/schedule';

@Module({ imports: [ScheduleModule.forRoot()] })
export class AppModule {}

@Injectable()
export class TasksService {
  @Cron('0 0 3 * * *')                 // 每天凌晨 3 点
  cleanUp() { /* ... */ }

  @Interval(60_000)                    // 每 60 秒
  heartbeat() { /* ... */ }

  @Timeout(5_000)                      // 启动后 5 秒执行一次
  init() { /* ... */ }
}
```

## 推荐目录结构与常用命令 / Project Structure

```text
my-nest-app/
├── src/
│   ├── main.ts               # 入口：全局管道/拦截器/过滤器、Swagger
│   ├── app.module.ts         # 根模块
│   ├── common/               # 通用：守卫、拦截器、过滤器、DTO
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── filters/
│   │   └── decorators/
│   ├── config/               # 配置
│   ├── modules/              # 业务模块
│   │   └── users/
│   │       ├── users.module.ts
│   │       ├── users.controller.ts
│   │       ├── users.service.ts
│   │       ├── entities/user.entity.ts
│   │       └── dto/create-user.dto.ts
│   └── shared/               # 共享服务（Redis、Prisma 等）
├── test/                     # e2e 测试
├── .env
└── package.json
```

!!! tip
    以「功能模块」组织目录，每个模块自带 `module / controller / service / dto / entity`，
    配合 `nest g resource` 可快速生成标准骨架，保持团队结构一致。
