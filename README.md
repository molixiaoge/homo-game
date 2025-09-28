# homo-game

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-8+-green.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.7.2-brightgreen.svg)](https://spring.io/projects/spring-boot)

## 简介

homo-game 是基于 homo-core 框架开发的游戏服务器解决方案，针对游戏开发场景设计的全响应式一站式分布式框架。该框架旨在实现一套游戏服务场景的通用解决方案，满足不同游戏设计场景需求，如 SLG、MMO、RPG、棋牌类游戏等。

### 核心特性

- 🚀 **全响应式编程**：基于 Reactor 的异步非阻塞编程模型
- 🏗️ **模块化架构**：清晰的模块分层，支持插件化扩展
- 🔄 **有状态服务**：支持实体路由和状态保持
- 🌐 **多协议支持**：HTTP、gRPC、TCP 统一抽象
- 💾 **多级存储**：Redis + MySQL + MongoDB 统一存储抽象
- 📊 **完整监控**：链路追踪、指标监控、日志聚合
- ⚡ **高性能**：连接池、对象池、批量处理优化

## 系统架构

![系统架构图](docs/images/架构图.jpg)

## 快速开始

### 环境要求

- Java 8+
- Maven 3.6+
- Redis 6.0+
- MySQL 8.0+
- Apollo 配置中心（可选）

### 安装依赖

```bash
# 克隆项目
git clone https://github.com/HZ00M/homo-game.git
cd homo-game

# 编译安装
mvn clean install -DskipTests
```

### 运行示例

```bash
# 启动登录服务
cd homo-game-login
mvn spring-boot:run

# 启动有状态代理
cd homo-game-stateful-proxy  
mvn spring-boot:run

# 启动持久化服务
cd homo-game-persistent-server
mvn spring-boot:run
```

## 核心模块

### 网关与代理层

#### homo-game-stateful-proxy
**有状态代理服务** - 处理长连接和实体路由

- **核心功能**：
  - TCP 长连接管理（基于 Netty）
  - 实体有状态路由（确保同一实体请求路由到同一实例）
  - 登录认证与重连处理
  - 消息序列化与协议编解码

- **核心流程**：
  ```
  客户端连接 → 登录认证 → 实体绑定 → 消息路由 → 后端服务调用
  ```

- **关键组件**：
  - `StatefulProxyLogicHandler`：消息处理逻辑
  - `ProxyGateServer`：网关服务器
  - `RouterHandler`：路由处理器

#### homo-game-proxy
**无状态代理服务** - HTTP/gRPC 转发代理

- **核心功能**：
  - HTTP/gRPC 请求转发
  - 服务发现与负载均衡
  - 限流熔断与重试机制
  - 协议转换与数据编解码

#### homo-common-proxy
**通用代理服务** - 流量治理与路由

- **核心功能**：
  - 统一限流熔断（基于 Sentinel）
  - 灰度发布与流量控制
  - 服务路由与负载均衡
  - 配置中心集成（Apollo）

### 业务服务层

#### homo-game-login
**登录认证服务** - 用户认证与会话管理

- **核心功能**：
  - 多渠道登录认证（第三方渠道/本地账号）
  - 会话管理与令牌颁发
  - 多端互踢与设备管理
  - 风控与安全策略

- **核心流程**：
  ```
  登录请求 → 渠道验证 → 用户认证 → 会话创建 → 令牌颁发 → 登录成功
  ```

- **关键接口**：
  - `auth()`：用户认证
  - `queryUserInfo()`：用户信息查询
  - `sendPhoneCode()`：发送验证码
  - `validatePhoneCode()`：验证码校验

#### homo-game-activity-core
**活动系统核心** - 活动规则与奖励管理

- **核心功能**：
  - 活动节点管理（Node 系统）
  - 事件驱动架构（Event 系统）
  - 进度跟踪与奖励发放
  - 多活动类型支持

- **核心流程**：
  ```
  活动配置 → 节点创建 → 事件订阅 → 进度更新 → 条件检查 → 奖励发放
  ```

- **关键组件**：
  - `Node`：活动节点抽象
  - `Owner`：数据所有者
  - `Component`：活动组件
  - `Event`：事件系统

### 数据存储层

#### homo-game-persistent-server
**持久化服务** - 数据落地与一致性保证

- **核心功能**：
  - 脏数据检测与标记
  - 批量数据落地（Redis → MySQL）
  - 版本控制与软删除
  - 失败重试与告警

- **核心流程**：
  ```
  数据变更 → 脏标记 → 定时扫描 → 批量落地 → 版本更新 → 清理标记
  ```

### 工具与门面层

#### homo-game-common-util
**通用工具模块** - 业务工具与系统集成

- **核心功能**：
  - 业务通用工具类
  - 系统集成封装
  - 响应式编程工具
  - 跨模块数据共享

#### 各 Facade 模块
**门面契约层** - 接口定义与协议生成

- **核心功能**：
  - 服务接口定义
  - Proto/gRPC 代码生成
  - 数据传输对象（DTO）
  - 错误码与异常定义

## 技术架构

### 分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    业务应用层                                │
├─────────────────────────────────────────────────────────────┤
│                    homo-core-facade                         │
│                   (门面抽象层)                              │
├─────────────────────────────────────────────────────────────┤
│  homo-core-utils  │  homo-core-properties  │  homo-core-system │
│    (工具模块)      │      (配置模块)        │     (系统模块)    │
├─────────────────────────────────────────────────────────────┤
│  homo-core-rpc    │  homo-core-gate       │  homo-core-mq     │
│   (RPC通信)       │     (网关)            │   (消息队列)      │
├─────────────────────────────────────────────────────────────┤
│  homo-core-storage │  homo-core-entity-ability │  homo-core-* │
│     (存储)        │      (实体能力)        │   (其他模块)     │
├─────────────────────────────────────────────────────────────┤
│                    Spring Boot 2.7.2                       │
│                    (基础框架)                              │
└─────────────────────────────────────────────────────────────┘
```

### 数据流转

```
客户端 → 有状态代理(TCP) → 后端服务（实体路由）
客户端 → 无状态代理(HTTP) → 后端服务（负载均衡）
登录成功 → MQ → 活动/画像等消费者
数据变更 → Dirty Redis → Persistent Server → MySQL
```

### 存储架构

```
L1: 内存缓存 (Caffeine)
    ↓ (缓存未命中)
L2: Redis 缓存
    ↓ (缓存未命中)  
L3: 持久化存储 (MySQL/MongoDB)
```

## 配置说明

### 基础配置

```yaml
# application.yml
homo:
  core:
    app-id: "homo-game"
    region-id: "region-1"
  rpc:
    grpc:
      server:
        port: 8080
  gate:
    tcp:
      port: 666
      boss-num: 1
      work-num: 4
  storage:
    redis:
      host: localhost
      port: 6379
    mysql:
      url: jdbc:mysql://localhost:3306/homo_game
      username: root
      password: password
```

### 服务发现配置

```yaml
homo:
  service:
    discovery:
      type: "consul"  # 或 eureka, nacos
      host: "localhost"
      port: 8500
```

## 开发指南

### 添加新服务

1. **创建 Facade 模块**：
   ```bash
   mkdir homo-game-new-service-facade
   # 定义接口和 Proto 文件
   ```

2. **实现服务模块**：
   ```bash
   mkdir homo-game-new-service
   # 实现业务逻辑
   ```

3. **注册服务**：
   ```java
   @ServiceExport(tagName = "newService:8080")
   public interface NewService {
       Homo<Response> process(Request request);
   }
   ```

### 添加新活动

1. **定义活动节点**：
   ```java
   @Component
   public class NewActivityNode extends Node {
       @Override
       public void onEvent(Event event) {
           // 处理活动事件
       }
   }
   ```

2. **配置活动规则**：
   ```java
   @EntityType(type = "activity")
   public class ActivityEntity extends BaseAbilityEntity<ActivityEntity> {
       // 活动实体实现
   }
   ```

## 监控与运维

### 关键指标

- **连接数**：在线用户数、连接建立/断开速率
- **请求量**：QPS、响应时间、错误率
- **存储**：缓存命中率、落地延迟、存储容量
- **业务**：登录成功率、活动参与率、奖励发放成功率

### 日志配置

```yaml
logging:
  level:
    com.homo: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

## 性能优化

### 连接优化

- 调整 Netty 线程池大小
- 优化连接池配置
- 启用 Epoll（Linux）

### 存储优化

- 合理设置缓存大小
- 优化批量操作
- 使用连接池

### 业务优化

- 避免阻塞操作
- 合理使用异步处理
- 优化数据库查询

## 故障排查

### 常见问题

1. **连接超时**：检查网络配置和超时设置
2. **内存泄漏**：检查对象引用和缓存配置
3. **数据不一致**：检查版本控制和并发处理
4. **性能下降**：检查资源使用和配置优化

### 调试工具

- 使用 Zipkin 进行链路追踪
- 通过 Micrometer 查看指标
- 利用日志分析问题

## 贡献指南

1. Fork 项目
2. 创建特性分支
3. 提交更改
4. 推送到分支
5. 创建 Pull Request

## 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情

## 依赖工程

- **homo-core**：核心框架
  - [GitHub](https://github.com/HZ00M/homo-core)
  - [Gitee](https://gitee.com/Hzoom/homo-core)

## 联系方式

- 项目维护者：HZ00M
- 邮箱：your-email@example.com
- 项目地址：https://github.com/HZ00M/homo-game