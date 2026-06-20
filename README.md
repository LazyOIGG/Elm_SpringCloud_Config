# Elm SpringCloud Config

本仓库是饿了么（Elm）Spring Cloud微服务架构的**配置中心仓库**，用于集中管理各微服务的配置文件。

## 适配后端项目

本配置仓库适配后端项目：[Elm_SpringCloud]

## 微服务列表

| 服务名称             | 配置文件                       | 端口    | 说明    |
|------------------|----------------------------|-------|-------|
| user-service     | `user-service-dev.yml`     | 15001 | 用户服务  |
| business-service | `business-service-dev.yml` | 15002 | 商户服务  |
| cart-service     | `cart-service-dev.yml`     | 15003 | 购物车服务 |
| order-service    | `order-service-dev.yml`    | 15004 | 订单服务  |

## 通用配置说明

所有微服务共享以下基础配置：

### 数据库配置
- **数据库类型**: MySQL
- **数据库名称**: elm
- **连接地址**: `jdbc:mysql://localhost:3306/elm`
- **连接池**: HikariCP
  - 最大连接数: 30
  - 最小空闲连接: 5

### JPA/Hibernate配置
- DDL自动更新: `update`
- SQL方言: `MySQLDialect`

### Knife4j (Swagger) 配置
- 启用状态: 开启
- 语言: 简体中文
- 认证:
  - 用户名: `admin`
  - 密码: `123456`

## 环境说明

当前配置文件为 **开发环境（dev）** 配置，文件命名规则为：
```
{服务名}-dev.yml
```

## 使用方式

1. 确保已安装并启动 Spring Cloud Config Server
2. 将本仓库配置为Config Server的Git远程仓库
3. 各微服务通过Config Server拉取配置

## 注意事项

⚠️ **安全提醒**: 配置文件中包含数据库密码等敏感信息，生产环境请：
- 使用环境变量或配置中心加密功能
- 不要将生产配置提交到公开仓库
