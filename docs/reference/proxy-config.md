# 代理配置

AgentSDK 在端侧通过**本地代理**承接业务流量。完成 `initialize` 后，三种代理模式按需取用：

| 模式 | 接口（概念名） | 适用场景 | 接入方式 |
|------|----------------|---------|---------|
| **TCP 代理** | `getLocalTCPProxy(host, port)` | Socket 编程、自定义协议、精细控制特定 host:port | 用返回的本地地址替换原始连接目标 |
| **HTTP 代理** | `getLocalHTTPProxy()` | OkHttp / URLSession / Dio 等 HTTP 客户端 | 作为 HTTP Proxy 注入客户端 |
| **SOCKS5 代理** | `getLocalSocks5Proxy()` | 非 HTTP 协议、通用代理需求 | 作为 SOCKS5 Proxy 注入 |

各模式的内部实现与三层代理决策机制请参阅 [架构总览 §3 代理模式](../introduction/architecture.md#3-代理模式)。

## 调用契约

- 所有代理接口在 `initialize` 成功（返回 `0`）后才可调用，否则返回未就绪。
- 返回的本地地址在 SDK 生命周期内**保持不变**，可缓存到业务层重复使用。
- TCP 代理是**按目标 host:port 注册**的：每个目标地址对应一个本地端口。重复调用同一 host:port 返回同一本地端口。

## 推荐：使用 axhttp 扩展

若应用使用主流 HTTP 框架，**不要手动调用代理接口**——直接使用 [支持的平台与框架](../guides/platforms.md#http-客户端扩展axhttp) 列出的 axhttp 扩展，业务层只需替换客户端实例。

## 各平台方法名

| 平台 | 取 HTTP 代理 | 取 SOCKS5 代理 | 取 TCP 代理 |
|------|--------------|---------------|-------------|
| Android | `AXService.getLocalHTTPProxy()` | `AXService.getLocalSocks5Proxy()` | `AXService.getLocalTCPProxy(host, port)` |
| iOS | `[AXService getLocalHTTPProxy]` | `[AXService getLocalSocks5Proxy]` | `[AXService getLocalTCPProxy:host port:port]` |
| Flutter | `AxService.getLocalHTTPProxy()` | `AxService.getLocalSocks5Proxy()` | `AxService.getLocalTCPProxy(host, port)` |

具体代码示例参见 [支持的平台与框架](../guides/platforms.md) 中各 Demo 仓库。

!!! note "更多 API 待补充"
    本页是首版骨架。后续将补充：返回结构定义（IP/端口 / 是否就绪 / 错误码）、并发与重入约束、自定义超时与拦截器注入、释放与销毁时序等。
