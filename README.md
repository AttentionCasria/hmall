# 黑马商城 (HmMall) 🛒

![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen.svg)
![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2022.x-blue.svg)
![MyBatis Plus](https://img.shields.io/badge/MyBatis%20Plus-3.5-blueviolet.svg)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3.x-orange.svg)
![Docker](https://img.shields.io/badge/Docker-latest-blue.svg)

**黑马商城** 是一个基于 **Spring Cloud 微服务架构** 打造的 B2C 电商解决方案。项目涵盖了从用户认证、商品搜索、购物车管理到订单处理、支付流程及物流追踪的全生命周期业务流程，旨在实现高可用、高并发的分布式电商系统。

---

## 🛠 详细技术栈 (Technology Stack)

### 1. 后端核心框架
* **核心框架**：Spring Boot 3.x（利用 AOT 编译优化及新特性，提升启动速度与运行效率）
* **微服务治理**：Spring Cloud Alibaba
    * **Nacos**：服务发现与注册中心、配置中心。
    * **Sentinel**：流量控制、熔断降级（保证系统在双 11 等高并发场景下的稳定性）。
    * **Gateway**：网关服务，负责统一路由、负载均衡及权限校验。
    * **OpenFeign**：声明式 HTTP 客户端，实现微服务间的优雅调用。
    * **Seata**：分布式事务解决方案，保证跨服务订单与库存操作的最终一致性。

### 2. 持久层与数据库
* **数据库**：MySQL 8.0（采用行级锁与索引优化，支持高并发读写）。
* **持久层框架**：MyBatis-Plus 3.5.x（利用其强大的 CRUD 抽象及 Lambda 表达式减少硬编码）。
* **连接池**：Druid / HikariCP（高性能数据库连接池）。

### 3. 中间件与缓存
* **分布式缓存**：Redis（用于存储 Token、商品热点信息、秒杀库存扣减，提升系统响应速度）。
* **消息队列**：RabbitMQ（实现订单超时自动关闭、库存异步解耦、物流状态通知）。
* **全文检索**：Elasticsearch 7.x / 8.x（实现千万级商品数据的实时倒排索引与分词搜索）。

### 4. 系统安全性与监控
* **身份验证**：JWT (JSON Web Token) + 拦截器，实现无状态分布式登录。
* **数据加解密**：BCrypt 强哈希算法存储用户密码。
* **链路追踪**：SkyWalking / Zipkin（可视化监控微服务间的调用链路及性能瓶颈）。

### 5. 运维与部署
* **容器化**：Docker & Docker Compose（实现环境的一键快速迁移与部署）。
* **服务器管理**：基于 Alibaba Cloud ECS 部署，使用 Nginx 进行动静分离与负载均衡。
* **管理面板**：使用 BT-Panel (宝塔) 与 FinalShell 进行服务器资源监控与自动化维护。

---

## 📦 模块拆分 (Modules)

```text
hmall
├── hm-common      // 核心公共包：工具类、枚举、异常拦截、R通用返回对象
├── hm-gateway     // Spring Cloud Gateway 网关：路由转发、JWT 校验、限流
├── hm-service     // 微服务业务模块
│   ├── user-service     // 用户模块：登录、注册、个人中心
│   ├── item-service     // 商品模块：分类查询、商品详情、库存管理
│   ├── cart-service     // 购物车模块：增删改查、价格校验
│   ├── order-service    // 订单模块：下单、延时任务处理、状态流转
│   └── pay-service      // 支付模块：对接微信/支付宝、支付回调处理
└── hm-api         // Feign Client 接口定义：解决服务间调用的循环依赖
```

---

## ✨ 核心业务场景展示

### 1. 分布式下单流程
集成 **Seata AT 模式**，在“创建订单”、“扣减库存”、“核销优惠券”三个微服务之间开启全局事务，确保订单支付环节的数据原子性。

### 2. 延时取消机制
利用 **RabbitMQ 死信队列 (TTL + DLX)** 实现订单超时处理。用户下单后 30 分钟未支付，系统自动触发取消逻辑并归还商品库存。

### 3. 高性能搜索优化
商品数据同步通过 **MQ 异步监听** 实现。当管理后台修改菜品或商品信息时，自动同步更新 Elasticsearch 索引，保证搜索结果的实时性与准确性。

---

## 🚀 快速启动

### 第一步：克隆项目
```bash
git clone https://github.com/AttentionCasria/hmall.git
```

### 第二步：环境配置
1.  **MySQL**: 执行 `sql` 目录下的脚本初始化数据库。
2.  **Nacos**: 启动 Nacos 服务，并将各模块的 `bootstrap.yml` 中的 `server-addr` 指向你的 Nacos 地址。
3.  **Redis/RabbitMQ**: 确保中间件已启动并可访问。

### 第三步：编译并运行
```bash
mvn clean install
# 依次启动 Gateway -> User -> Item -> Cart -> Order
```

---

## 💡 个人思考与优化

* **性能提升**：通过 Redis 预热热点商品数据，QPS 提升了约 40%。
* **架构解耦**：将公共 POJO 和 Feign 接口独立成 `hm-api` 模块，极大减少了模块间的代码冗余。
* **防御编程**：在秒杀/高频接口增加了 Sentinel 热点参数限流，有效防止了恶意刷单行为。

---

> **致谢**：感谢黑马程序员提供的项目原型与指导。如果您觉得本项目对您有帮助，请给个 **Star** 🌟！
