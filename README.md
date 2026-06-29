# Elm SpringCloud Config

本仓库是饿了么（Elm）Spring Cloud 微服务架构的**远程配置中心仓库**，用于集中管理各微服务的开发环境配置。

适配后端项目：[Elm_SpringCloud](https://github.com/LazyOIGG/Elm_SpringCloud)

## 配置文件列表

| 配置文件                       | 对应服务集群               | 说明    |
|---------------------------|----------------------|-------|
| `service-business-dev.yml` | Business (11000-11002) | 商家服务  |
| `service-cart-dev.yml`     | Cart (12000-12002)     | 购物车服务 |
| `service-order-dev.yml`    | Order (13000-13002)    | 订单服务  |
| `service-user-dev.yml`     | User (14000-14002)     | 用户服务  |

## 配置说明

### 端口配置

端口不在远程配置中定义，由各实例的本地 `bootstrap.yml` 指定，避免集群实例端口冲突：

| 服务      | 实例1  | 实例2  | 实例3  |
|---------|------|------|------|
| Business | 11000 | 11001 | 11002 |
| Cart     | 12000 | 12001 | 12002 |
| Order    | 13000 | 13001 | 13002 |
| User     | 14000 | 14001 | 14002 |

### 数据库配置

- **数据库类型**：MySQL
- **数据库名称**：`elm`
- **连接地址**：`jdbc:mysql://localhost:3306/elm`
- **连接池**：HikariCP（最大连接数 30，最小空闲 5）

### JPA 配置

- DDL 策略：`validate`（启动时校验表结构）
- SQL 方言：`MySQLDialect`
- 命名策略：`PhysicalNamingStrategyStandardImpl`

### Feign 与 LoadBalancer 配置

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true
      client:
        config:
          default:
            connectTimeout: 3000   # 连接超时 3 秒
            readTimeout: 5000      # 读取超时 5 秒
    loadbalancer:
      health-check:
        path:
          default: /actuator/health # 健康检查路径
        interval: 10s              # 检查间隔
      cache:
        enabled: true
        ttl: 30s                   # 服务列表缓存时间
```

`service-cart-dev.yml` 和 `service-order-dev.yml` 还包含自定义负载均衡策略配置：

```yaml
elm:
  loadbalancer:
    strategy: three-times          # 可选：three-times、random
```

### Resilience4j 容错配置

各服务配置了 CircuitBreaker、RateLimiter、Bulkhead 和 TimeLimiter。当前开发环境中，商家服务限流阈值为 100 次 / 2 秒，用户、购物车、订单服务限流阈值为 20 次 / 2 秒；隔板最大并发为 20，等待时间为 0 秒。

### 动态配置测试项

各配置文件包含 `custom.message`，业务服务通过 `ConfigMessageProvider` 读取该值，可配合 `/configMessage` 接口和 `busrefresh` 验证 Spring Cloud Bus 动态刷新。

### Knife4j 配置

- 启用状态：开启
- 语言：简体中文
- Basic 认证：用户名 `admin`，密码 `123456`

## 环境说明

当前为 **开发环境（dev）** 配置，文件命名规则：

```
service-{服务名}-dev.yml
```

## 使用方式

1. 配置中心（Config Server）从本 Git 仓库读取配置
2. 各微服务启动时通过 Config Server 拉取对应配置
3. 通过 Spring Cloud Bus（RabbitMQ）实现配置动态刷新

刷新示例：

```bash
curl -X POST http://localhost:20000/actuator/busrefresh
```

## 注意事项

- 配置文件中包含数据库密码等本地开发凭据，请按本机 MySQL 配置修改，**不要将真实生产凭据提交到公开仓库**
- 生产环境请使用环境变量或配置中心加密功能管理敏感信息
