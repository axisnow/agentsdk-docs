# 支持的平台与框架

每个平台/框架都提供独立的 Demo 工程，包含可运行示例、依赖配置和平台特有注意事项（Android ProGuard、iOS Bitcode/ATS 等）。直接点对应 GitHub 链接获取完整代码。

## 核心 SDK 与 axhttp 分层关系

```
┌──────────────────────────────────────────────┐
│            应用层（App Code）                  │
├──────────────────────────────────────────────┤
│   axhttp 扩展（可选）                          │
│   OkHttp / Retrofit / URLSession / AxClient   │
├──────────────────────────────────────────────┤
│   核心 SDK（必选）                             │
│   本地代理 · 加密隧道 · DNS 解析 · 认证        │
└──────────────────────────────────────────────┘
```

- 所有平台都需要**先集成核心 SDK**
- 若应用使用主流 HTTP 框架（OkHttp / URLSession / `http` 包等），**推荐使用对应的 axhttp 扩展**以简化接入
- 若需要代理**非 HTTP 协议**或有自定义需求，直接使用核心 SDK 的代理接口即可

## 平台支持矩阵

| 平台 | 语言 | 最低版本 | Demo 仓库 |
|------|------|---------|----------|
| Android | Java / Kotlin | API 21 (Android 5.0) | [sdk-quickstarts / android](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/android/demo/) |
| iOS | Objective-C / Swift | iOS 12.0 | [sdk-quickstarts / ios](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/ios/demo/) |
| Flutter | Dart | Flutter 2.0+ | [sdk-quickstarts / flutter](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/flutter/demo/) |

完整的安装与构建步骤见各 Demo 仓库的 README。

## axhttp 框架覆盖

axhttp 是可选的零侵入接入层，与主流 HTTP 框架配套使用，业务层无需手动获取代理地址。

### Android

| 框架 | 语言 | Demo 仓库 |
|------|------|----------|
| OkHttp | Java | [okhttp-java](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/android/okhttp/java-okhttp/demo/) |
| OkHttp | Kotlin | [okhttp-kotlin](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/android/okhttp/kotlin-okhttp/demo/) |
| Retrofit | Java | [retrofit-java](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/android/retrofit/java-retrofit/demo/) |
| Retrofit | Kotlin | [retrofit-kotlin](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/android/retrofit/kotlin-retrofit/demo/) |
| HttpsURLConnection | Java | [httpsurlconnection-java](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/android/httpsurlconn/java-httpsurlconn/demo/) |

### iOS

| 框架 | 语言 | Demo 仓库 |
|------|------|----------|
| URLSession | Objective-C | [urlsession-objc](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/ios/urlsession/oc-urlsession/demo) |
| URLSession | Swift | [urlsession-swift](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/axhttp/ios/urlsession/swift-urlsession/demo) |

### Flutter

Flutter 插件**内置** HTTP 客户端封装，无需单独的 axhttp 包：

| 类 | 对接框架 | 说明 |
|------|---------|------|
| `AxClient` | [http](https://pub.dev/packages/http) 包 | `http.Client` 的直接替换，推荐方式 |
| `AxHttpClient` | `dart:io` `HttpClient` | `HttpClient` 的直接替换，Dio 可通过此类集成 |

详见 Flutter Demo：[sdk-quickstarts / flutter](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/flutter/demo/)。

## 何时使用 axhttp vs 核心 SDK

- **应用使用主流 HTTP 框架** → 直接选对应的 axhttp 扩展，最少代码完成接入
- **应用使用自定义网络栈 / 非 HTTP 协议** → 用核心 SDK 的 `getLocalHTTPProxy()` / `getLocalSocks5Proxy()`，详见 [集成网络库 / 手动调用代理 API](network-integration.md#手动调用代理-api) 与 [参考 / 代理配置](../reference/proxy-config.md)
