---
title: go-microservice-evolution
date: Mon May 25 2026 08:00:00 GMT+0800 (China Standard Time)
status: published
published_at: 2026-05-25T03:37:55+08:00
updated_at: 2026-05-25T03:37:55+08:00
---

# 从零搭建一个 Go 微服务：从单体到 gRPC 的演进之路

单体架构是项目的起点，也是最终需要被拆解的原罪。这篇文章记录我把一个简单的 Go HTTP 服务逐步演进为 gRPC 微服务的过程。

## 为什么从单体开始

过早拆分微服务会引入不必要的复杂性：

- 分布式事务变得棘手
- 调试需要跨多个服务追踪
- 部署和运维成本显著增加
- 网络延迟成为新的瓶颈

从单体开始，当某个模块的性能或团队协作成为瓶颈时再拆，这是更务实的做法。

## 演进路线

### 第一阶段：单体 HTTP 服务

```go
// main.go - 最初的简单结构
func main() {
    r := gin.Default()
    r.GET("/users/:id", getUser)
    r.POST("/orders", createOrder)
    r.Run(":8080")
}
```

所有路由、业务逻辑、数据访问都在一个包里。对于早期阶段来说这完全够用——部署简单，调试方便。

### 第二阶段：按领域拆分包

当代码量增长到每个文件超过 500 行时，我开始按业务域拆分包结构：

```
.
├── cmd/server/main.go
├── internal/
│   ├── user/          # 用户模块
│   ├── order/         # 订单模块
│   ├── product/       # 商品模块
│   └── payment/       # 支付模块
└── pkg/
    └── database/      # 共享的数据库工具
```

### 第三阶段：引入 gRPC

当另一个团队需要用 Rust 写一个推荐服务，需要调用用户服务的接口时，REST API 的局限性开始显现：

```protobuf
service UserService {
    rpc GetUser(GetUserRequest) returns (User);
    rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
    rpc UpdateUser(UpdateUserRequest) returns (User);
}
```

gRPC 带来的好处：
- 强类型契约，自动生成客户端代码
- 原生支持流式传输
- 基于 HTTP/2 的多路复用
- Protocol Buffers 序列化比 JSON 更高效

## 关键经验

1. **服务拆分的信号不是代码行数，而是团队规模和部署频率**。当两个团队需要独立部署时，拆分才有意义。
2. **数据库是最后拆分的**。共享数据库在早期减少了大量一致性问题的复杂度。
3. **API 网关要尽早引入**。即使只有两个服务，统一的入口也能简化客户端逻辑和认证鉴权。

这段演进之路没有银弹，每一步都是在具体的业务约束下做出的权衡。
