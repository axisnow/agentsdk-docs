# 平台与框架适配

每个平台/框架都提供独立的 Demo 工程，包含可运行示例、依赖配置和平台特有注意事项（Android ProGuard、iOS Bitcode/ATS 等）。直接点对应 GitHub 链接获取完整代码。

!!! warning "Demo 链接为占位 URL"
    本页所有 Demo 链接当前指向占位 `https://github.com/axisnow/agentsdk-demos/...`。
    待对外 demo 仓库地址确定后会批量替换。

axhttp 与核心 SDK 的分层关系见 [概览 / 核心 SDK 与 axhttp 的分层关系](../introduction/overview.md#核心-sdk-与-axhttp-的分层关系)。

## 平台 SDK（核心库）

| 平台 | 语言 | 最低版本 | Demo 仓库 |
|------|------|---------|----------|
| Android | Java | API 21 (Android 5.0) | [agentsdk-demos / android](https://github.com/axisnow/agentsdk-demos/tree/main/packing/android/demo) |
| iOS | Objective-C | iOS 12.0 | [agentsdk-demos / ios](https://github.com/axisnow/agentsdk-demos/tree/main/packing/ios/demo) |
| Flutter | Dart | Flutter 2.0+ | [agentsdk-demos / flutter](https://github.com/axisnow/agentsdk-demos/tree/main/packing/flutter/demo) |
| Unity | C# | Unity 2020 LTS+ | [agentsdk-demos / unity](https://github.com/axisnow/agentsdk-demos/tree/main/packing/unity/plugin) |

## HTTP 客户端扩展（axhttp）

可选的零侵入接入层，与主流 HTTP 框架配套使用，业务层无需手动获取代理地址。

### Android

| 框架 | 语言 | 最低版本 | Demo 仓库 |
|------|------|---------|----------|
| OkHttp | Java | API 21 | [okhttp-java](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/android/okhttp/java-okhttp/demo) |
| OkHttp | Kotlin | API 21 | [okhttp-kotlin](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/android/okhttp/kotlin-okhttp/demo) |
| Retrofit | Java | API 21 | [retrofit-java](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/android/retrofit/java-retrofit/demo) |
| Retrofit | Kotlin | API 21 | [retrofit-kotlin](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/android/retrofit/kotlin-retrofit/demo) |
| HttpsURLConnection | Java | API 21 | [httpsurlconnection-java](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/android/httpsurlconn/java-httpsurlconn/demo) |

### iOS

| 框架 | 语言 | 最低版本 | Demo 仓库 |
|------|------|---------|----------|
| URLSession | Objective-C | iOS 12.0 | [urlsession-objc](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/ios/urlsession/oc-urlsession/demo) |
| URLSession | Swift | iOS 12.0 | [urlsession-swift](https://github.com/axisnow/agentsdk-demos/tree/main/axhttp/ios/urlsession/swift-urlsession/demo) |

### Flutter

Flutter 插件**内置** HTTP 客户端封装，无需单独的 axhttp 包：

| 类 | 对接框架 | 说明 |
|------|---------|------|
| `AxClient` | [http](https://pub.dev/packages/http) 包 | `http.Client` 的直接替换，推荐方式 |
| `AxHttpClient` | `dart:io` `HttpClient` | `HttpClient` 的直接替换，Dio 可通过此类集成 |

详见 Flutter Demo：[agentsdk-demos / flutter](https://github.com/axisnow/agentsdk-demos/tree/main/packing/flutter/demo)。

## 何时使用 axhttp vs 核心 SDK

- **应用使用主流 HTTP 框架** → 直接选对应的 axhttp 扩展，最少代码完成接入
- **应用使用自定义网络栈 / 非 HTTP 协议** → 用核心 SDK 的 `getLocalHTTPProxy()` / `getLocalSocks5Proxy()` / `getLocalTCPProxy()`，详见 [集成网络库 / 手动调用代理 API](network-integration.md#手动调用代理-api) 与 [参考 / 代理配置](../reference/proxy-config.md)
