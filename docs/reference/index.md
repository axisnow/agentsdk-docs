# 参考

本节给出 AgentSDK 的查阅类内容：API 定义、错误码、排障流程、术语表。

## 章节地图

### API 概念（跨平台一致）

手写的概念层 API 定义——参数语义、默认值、约束、返回值。不涉及具体平台的字段名 / 大小写 / 嵌套形式。

- [初始化参数](init-parameters.md) — `initialize` 接受的全部参数：AccessKey、Edge 地址列表、加密隧道开关、DNS 配置
- [代理配置](proxy-config.md) — 本地代理获取接口（HTTP / SOCKS5 / TCP）的语义与适用场景

### 各平台 API

由各平台 doc generator 自动生成的方法签名、字段名、返回类型——查具体 API 写代码时看这里。

- [Android](platform-api/android.md) — Javadoc / Dokka 产物（建设中）
- [iOS](platform-api/ios.md) — DocC / Jazzy 产物（建设中）
- [Flutter](platform-api/flutter.md) — `dart doc` 产物（建设中）
- [Unity](platform-api/unity.md) — DocFX 产物（建设中）

### 诊断

- [错误码](../errors.md) — 所有 API 错误码定义（按通用 / 初始化 / DNS / Proxy 分段）
- [排障](../troubleshooting.md) — 排障决策树、常见问题、平台特有问题、最佳实践

> 跨文档关键术语集中定义见 [概念与架构 / 术语表](../glossary.md)。

## API 命名约定

- 不同平台的 SDK 在方法命名上保持语义一致：
    - **`initialize`**（Android / iOS）/ **`init`**（Flutter / Unity）
    - **`getLocalHTTPProxy`** / **`getLocalSocks5Proxy`** / **`getLocalTCPProxy`**
- 各平台的实际方法签名与字段名以 [各平台 API](platform-api/index.md) 或 [Demo 仓库](../guides/platforms.md) 的代码为准。
