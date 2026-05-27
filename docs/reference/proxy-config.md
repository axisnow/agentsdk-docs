# 代理配置

AxisNow SDK 在端侧通过**本地代理**承接业务流量。完成 `initialize` 后，两种代理模式按需取用：

| 模式 | 接口（概念名） | 适用场景 | 接入方式 |
|------|----------------|---------|---------|
| **HTTP 代理** | `getLocalHTTPProxy()` | OkHttp / URLSession / Dio 等 HTTP 客户端 | 作为 HTTP Proxy 注入客户端 |
| **SOCKS5 代理** | `getLocalSocks5Proxy()` | 非 HTTP 协议、通用代理需求 | 作为 SOCKS5 Proxy 注入 |

端到端接入模式（Secure Proxy / Standard Proxy）的工作原理见 [白皮书 / 请求链路](../introduction/whitepaper.md#请求链路)。

## 调用契约

- 所有代理接口在 `initialize` 成功（返回 `0`）后才可调用，否则返回未就绪。
- 返回的本地地址在 SDK 生命周期内**保持不变**，可缓存到业务层重复使用。

## 推荐：使用 axhttp 扩展

若应用使用主流 HTTP 框架，**不要手动调用代理接口**——直接使用 [指南 / 支持的平台与框架 / axhttp 框架覆盖](../guides/platforms.md#axhttp-框架覆盖) 列出的 axhttp 扩展，业务层只需替换客户端实例。

## 各平台方法名

| 平台 | 取 HTTP 代理 | 取 SOCKS5 代理 |
|------|--------------|---------------|
| Android | `AXService.getLocalHTTPProxy()` | `AXService.getLocalSocks5Proxy()` |
| iOS | `[AXService getLocalHTTPProxy]` | `[AXService getLocalSocks5Proxy]` |
| Flutter | `AxService.getLocalHTTPProxy()` | `AxService.getLocalSocks5Proxy()` |

具体代码示例参见 [指南 / 集成网络库](../guides/network-integration.md) 与各 Demo 仓库链接（见 [指南 / 支持的平台与框架](../guides/platforms.md)）。

!!! note "更多 API 待补充"
    本页是首版骨架。后续将补充：返回结构定义（IP/端口 / 是否就绪 / 错误码）、并发与重入约束、自定义超时与拦截器注入、释放与销毁时序等。
