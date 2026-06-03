# 集成网络库

初始化完成后，把业务请求接入 SDK 的**本地代理**。SDK 对外提供两种代理模式，按业务网络库选其一：

| 模式 | API（概念名） | 适用场景 | 接入方式 |
|------|----------------|---------|---------|
| **HTTP 代理** | `getLocalHTTPProxy()` | OkHttp / URLSession / Dio 等 HTTP 客户端 | 作为 HTTP Proxy 注入客户端 |
| **SOCKS5 代理** | `getLocalSocks5Proxy()` | 非 HTTP 协议、通用代理需求 | 作为 SOCKS5 Proxy 注入 |

完整接口定义见 [参考 / 代理配置](../reference/proxy-config.md)；端到端接入模式（Secure Proxy / Standard Proxy）的工作原理见 [白皮书 / 请求链路](../introduction/whitepaper.md#请求链路)。

## 推荐：使用 axhttp 扩展（零侵入）

如果业务使用主流 HTTP 框架（OkHttp / Retrofit / URLSession / `http` 包等），**直接用对应的 axhttp 扩展**——它封装了上述代理调用，业务层只需替换客户端实例：

=== "Android (Java + OkHttp)"

    ```java
    import com.axsecurity.sdk.axhttp.okhttp.AXHTTPService;

    // 直接取 SDK 托管的 OkHttpClient
    OkHttpClient client = AXHTTPService.getOkHttpClient();

    Request request = new Request.Builder()
        .url("https://api.example.com/data")
        .build();

    try (Response response = client.newCall(request).execute()) {
        // 正常使用
    }
    ```

    如需自定义超时、拦截器、SSL/TLS：

    ```java
    AXHTTPService.setOkHttpClientBuilder(
        new OkHttpClient.Builder()
            .connectTimeout(5, TimeUnit.SECONDS)
            .readTimeout(10, TimeUnit.SECONDS)
    );
    OkHttpClient client = AXHTTPService.getOkHttpClient();
    ```

=== "iOS (Objective-C + URLSession)"

    ```objc
    #import <AXSecurityNSURLSession/AXNSURLSessionHeader.h>

    NSURLSessionConfiguration *cfg = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *session = [AXNSURLSession sessionWithConfiguration:cfg];

    NSURL *url = [NSURL URLWithString:@"https://api.example.com/data"];
    NSURLSessionDataTask *task =
        [session dataTaskWithURL:url
               completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
            // 正常使用
        }];
    [task resume];
    ```

=== "Flutter (Dart + http)"

    ```dart
    import 'package:axsecurity_flutter_plugin/axsecurity_flutter_plugin.dart';
    import 'package:http/http.dart' as http;

    final http.Client client = AxClient();
    final response = await client.get(Uri.parse('https://api.example.com/data'));
    // 正常使用
    ```

各平台完整支持矩阵与 Demo 仓库见 [支持的平台与框架](platforms.md)。

## 手动调用代理 API

如果框架不在 axhttp 覆盖范围内，或需要非 HTTP 协议代理，直接调用核心 SDK 的代理接口。两种代理按需选一即可，无需同时配置。

### HTTP 代理

适用于 HTTP / HTTPS 流量。

=== "Android (Java)"

    ```java
    import com.axsecurity.sdk.service.AXService;
    import com.axsecurity.sdk.service.AXLocalProxy;
    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.Response;
    import java.net.InetSocketAddress;
    import java.net.Proxy;

    AXLocalProxy http = AXService.getLocalHTTPProxy();
    Proxy httpProxy = new Proxy(Proxy.Type.HTTP,
            new InetSocketAddress(http.getIp(), http.getPort()));

    // 注入到 OkHttpClient 后发起请求
    OkHttpClient client = new OkHttpClient.Builder()
            .proxy(httpProxy)
            .build();

    Request request = new Request.Builder()
            .url("https://api.example.com/data")
            .build();

    try (Response response = client.newCall(request).execute()) {
        String body = response.body().string();
        // 处理响应
    }
    ```

=== "iOS (Objective-C)"

    ```objc
    #import <AXSecurity/axsecurity.h>

    // 同时配置 HTTP 与 HTTPS 两组键，否则 https:// 请求会绕开代理
    AXLocalProxy *http = [AXService getLocalHTTPProxy];
    NSURLSessionConfiguration *cfg = [NSURLSessionConfiguration defaultSessionConfiguration];
    cfg.connectionProxyDictionary = @{
        @"HTTPEnable":  @YES,
        @"HTTPProxy":   http.ip,
        @"HTTPPort":    @(http.port),
        @"HTTPSEnable": @YES,
        @"HTTPSProxy":  http.ip,
        @"HTTPSPort":   @(http.port),
    };

    // 用配置好的 cfg 创建 session 并发起请求
    NSURLSession *session = [NSURLSession sessionWithConfiguration:cfg];
    NSURL *url = [NSURL URLWithString:@"https://api.example.com/data"];
    NSURLSessionDataTask *task =
        [session dataTaskWithURL:url
               completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
            if (error) {
                NSLog(@"request failed: %@", error);
                return;
            }
            // 处理响应
        }];
    [task resume];
    ```

=== "Flutter (Dart)"

    ```dart
    import 'dart:convert';
    import 'dart:io';
    import 'package:axsecurity_flutter_plugin/axsecurity_flutter_plugin.dart';

    final http = await AxService.getLocalHTTPProxy();
    if (http == null) {
      // SDK 未就绪，检查初始化结果或稍后重试
      return;
    }

    final httpClient = HttpClient();
    httpClient.findProxy = (uri) => 'PROXY ${http.ip}:${http.port}';

    // 用配置好的 httpClient 发起请求
    final request = await httpClient.getUrl(Uri.parse('https://api.example.com/data'));
    final response = await request.close();
    final body = await response.transform(utf8.decoder).join();
    // 处理响应
    ```

### SOCKS5 代理

适用于非 HTTP 协议或需要在 TCP 层统一接管的场景。

=== "Android (Java)"

    ```java
    import com.axsecurity.sdk.service.AXService;
    import com.axsecurity.sdk.service.AXLocalProxy;

    AXLocalProxy socks5 = AXService.getLocalSocks5Proxy();
    Proxy socksProxy = new Proxy(Proxy.Type.SOCKS,
            new InetSocketAddress(socks5.getIp(), socks5.getPort()));
    // 注入到网络客户端
    ```

=== "iOS (Objective-C)"

    ```objc
    #import <AXSecurity/axsecurity.h>

    AXLocalProxy *socks5 = [AXService getLocalSocks5Proxy];
    NSURLSessionConfiguration *cfg = [NSURLSessionConfiguration defaultSessionConfiguration];
    cfg.connectionProxyDictionary = @{
        @"SOCKSEnable": @YES,
        @"SOCKSProxy":  socks5.ip,
        @"SOCKSPort":   @(socks5.port),
    };
    ```

=== "Flutter (Dart)"

    `dart:io` 的 `HttpClient.findProxy` 仅支持 `PROXY` / `DIRECT`，不原生支持 SOCKS5。若需 SOCKS5 转发，请参考 Flutter Demo 中的 socket 级集成方式。

## WebView 接入

如果业务页面运行在系统 WebView 中（Android `WebView` / iOS `WKWebView`），用 SDK 提供的 WebView 封装层把 WebView 的**全部流量**（主文档导航 + 页面子资源 + JS `fetch`/`XHR`/`WebSocket`）经 SDK 本地 **HTTP 代理**转发——无需逐个请求改造页面代码。

封装层不需要单独初始化：核心 SDK 仍由 `AXService.initialize(...)` 启动，封装只在接入时按需读取其本地代理（与 axhttp 一致的无 init 模型）。**接入前请先确保 `AXService.initialize(...)` 成功返回 `0`**。

所有平台的入口都返回 **`int` 错误码**：`0` 表示成功（已走 SDK 代理），负数表示代理未启用（WebView 降级直连，原因记录到平台日志）。错误码与 SDK 统一码表（`pkg/sdkerr/code.go`）一致。封装均**不抛异常**，失败时已自动让 WebView 走直连，页面照常打开。

=== "Android (Java)"

    封装包 `com.axsecurity.sdk.webview`，入口 `AXWebViewService.installOnWebView(webView, base)`：它**包装**你已有的 `WebViewClient`（回调照常转发）、把包装后的 client 挂到 WebView 上，并启用 SDK 的**进程级全局代理**。首次导航请用 `AXWebViewService.load(webView, url)` 发起。

    ```java
    import com.axsecurity.sdk.webview.AXWebViewService;

    // base 传入你已有的 WebViewClient；没有自己的 client 时传 null
    int rc = AXWebViewService.installOnWebView(webView, myWebViewClient);
    if (rc != 0) {
        // WebView 降级直连，原因见 logcat（tag "AXSDK"）；错误码见下表
    }

    // 发起首次导航：用 load(...) 而非直接 webView.loadUrl(...)，见下方「首次加载时序」
    AXWebViewService.load(webView, url);
    ```

    | 返回码 | 含义 | 处理 |
    |------|------|------|
    | `0` | 成功，已走 SDK 代理 | — |
    | `-2` | 参数非法（`webView` 为 null） | 传入有效的 WebView |
    | `-101` | SDK 未初始化 / 本地代理不可用 | 确认 `AXService.initialize(...)` 返回 `0` 后再调一次（可重复调用） |
    | `-401` | 设备 WebView 不支持 `PROXY_OVERRIDE` | 升级系统 WebView 组件；无法升级的设备降级直连 |

    !!! warning "首次加载时序"

        `installOnWebView` 返回 `0` 只表示代理设置**已发起**——底层 `ProxyController.setProxyOverride(...)` 是**异步**的，代理在其 completion 回调后才真正生效。若紧接着直接 `webView.loadUrl(...)`，**首次导航**有概率在代理生效前发出、走直连绕过 SDK（冷启动时最易命中）。请用 `AXWebViewService.load(webView, url)` 发起首次加载：它会在代理生效后才真正 `loadUrl`；代理不可用或回调超时时降级直连立即加载，保证页面不空白。竞态只存在于首次加载，后续导航无此问题。

    !!! warning "其它注意事项"

        - **不要在 `installOnWebView` 之后再自己调用 `setWebViewClient(...)`**——封装已接管 client 的设置，再设会覆盖 SDK 的 client。
        - **可重复调用**：若 WebView 在 SDK 初始化前已构造，可在 `AXService.initialize(...)` 成功后再调一次，把该 WebView 升级到走 SDK。
        - **进程级全局**：成功后影响 App 内所有 WebView，无法只对单个实例生效；代理在调用时即生效。
        - `PROXY_OVERRIDE` 跟随可独立升级的 WebView 组件（而非系统 API level）；极少数无法更新 WebView 的设备会降级直连。
        - **不支持混合框架**：Cordova / Capacitor / React Native / Flutter 的 `WebViewClient` 由框架持有，无法使用 `installOnWebView`。

=== "iOS (Objective-C)"

    封装框架 `AXSecurityWebView`，入口 `[AXWebViewService installOnWebView:]`：把 SDK 本地 HTTP 代理写入该 WebView 的 `WKWebsiteDataStore`，**不改动你的 `WKWebViewConfiguration`、`navigationDelegate` 或任何其它配置**。

    ```objc
    #import <AXSecurityWebView/AXSecurityWebViewHeader.h>

    // 你自己创建的 WebView（可带自己的 delegate / 配置）
    WKWebView *webView = ...;

    if (@available(iOS 17.0, *)) {
        int rc = [AXWebViewService installOnWebView:webView];
        if (rc != 0) {
            NSLog(@"代理未生效，错误码: %d", rc);  // 例如 -101：SDK 尚未初始化
        }
    }

    // 安装之后再加载，代理对本次及后续导航生效
    [webView loadRequest:[NSURLRequest requestWithURL:
        [NSURL URLWithString:@"https://your.page.example.com/"]]];
    ```

    | 返回码 | 含义 | 处理方式 |
    |--------|------|---------|
    | `0` | 成功，已走 SDK 代理 | — |
    | `-2` | 传入的 WebView 为 nil | 传入有效的 `WKWebView` |
    | `-101` | SDK 未初始化 / 本地代理不可用 | 确认 `[AXService initialize:]` 返回 `0` 后再调一次（可重复调用） |
    | `-411` | 无法基于本地代理端点构造 `nw_proxy_config` | 检查 SDK 状态与 Edge 连通性 |

    !!! warning "注意事项"

        - **仅支持 iOS 17.0+**：WebView 接入依赖 iOS 17 起提供的 `WKWebsiteDataStore.proxyConfigurations`；更低系统上 WebView 流量保持直连（SDK 核心本身最低支持 iOS 12.0）。
        - **安装时机**：在 `[AXService initialize:]` 返回 `0` **之后**、首次 `loadRequest:` **之前**安装；若加载后才安装，请随后 `reload` 一次。
        - **刷新**：每次调用都会重新向 SDK 拉取当前代理地址，需要刷新时再次调用即可（无单独刷新 API）。
        - **代理作用域跟随 dataStore**：用 `[WKWebsiteDataStore nonPersistentDataStore]` 或自建 store → 仅作用于该 WebView（推荐，隔离清晰）；用默认持久化 store → 影响所有使用默认 store 的 WebView。`WKWebView` 的 store 创建后不可更换。

    所需链接的系统库（除两个 `xcframework` 外）：`WebKit`、`Network`、`DeviceCheck`、`CoreTelephony`，以及 `libz.tbd` / `libc++.tbd` / `libresolv.tbd`。

=== "iOS (Swift)"

    同一个 `AXSecurityWebView` 框架，Objective-C 的 `installOnWebView:` 在 Swift 中即 `AXWebViewService.installOnWebView(_:)`，返回 `Int32` 错误码（与 Objective-C 一致，**不抛异常**）：把 SDK 本地 HTTP 代理写入该 WebView 的 `WKWebsiteDataStore`，**不改动你的 `WKWebViewConfiguration`、`navigationDelegate` 或任何其它配置**。

    ```swift
    import AXSecurityWebView

    // 你自己创建的 WebView（可带自己的 delegate / 配置）
    let webView: WKWebView = ...

    if #available(iOS 17.0, *) {
        let rc = AXWebViewService.installOnWebView(webView)
        if rc != 0 {
            NSLog("代理未生效，错误码: \(rc)")  // 例如 -101：SDK 尚未初始化
        }
    }

    // 安装之后再加载，代理对本次及后续导航生效
    if let url = URL(string: "https://your.page.example.com/") {
        webView.load(URLRequest(url: url))
    }
    ```

    | 返回码 | 含义 | 处理方式 |
    |--------|------|---------|
    | `0` | 成功，已走 SDK 代理 | — |
    | `-2` | 传入的 WebView 为 nil | 传入有效的 `WKWebView` |
    | `-101` | SDK 未初始化 / 本地代理不可用 | 确认 `AXService.initialize(_:)` 返回 `0` 后再调一次（可重复调用） |
    | `-411` | 无法基于本地代理端点构造 `nw_proxy_config` | 检查 SDK 状态与 Edge 连通性 |

    !!! warning "注意事项"

        - **仅支持 iOS 17.0+**：WebView 接入依赖 iOS 17 起提供的 `WKWebsiteDataStore.proxyConfigurations`；更低系统上 WebView 流量保持直连（SDK 核心本身最低支持 iOS 12.0）。
        - **安装时机**：在 `AXService.initialize(_:)` 返回 `0` **之后**、首次 `load(_:)` **之前**安装；若加载后才安装，请随后 `reload()` 一次。
        - **刷新**：每次调用都会重新向 SDK 拉取当前代理地址，需要刷新时再次调用即可（无单独刷新 API）。
        - **代理作用域跟随 dataStore**：用 `.nonPersistent()` 或自建 store → 仅作用于该 WebView（推荐，隔离清晰）；用默认持久化 store → 影响所有使用默认 store 的 WebView。`WKWebView` 的 store 创建后不可更换。

    所需链接的系统库（除两个 `xcframework` 外）：`WebKit`、`Network`、`DeviceCheck`、`CoreTelephony`，以及 `libz.tbd` / `libc++.tbd` / `libresolv.tbd`。

WebView 封装层当前仅提供代理转发，尚不支持对请求做签名或改写头部。完整接入步骤与可运行示例见各平台 WebView Demo（[Android](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/android/demo/webview-demo/) / [iOS Objective-C](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/ios/demo/) / [iOS Swift](https://github.com/axisnow/sdk-quickstarts/tree/HEAD/ios/swift-demo/)）。

## 调用约束

- 所有代理接口在 `initialize` 成功（返回 `0`）后才可调用，否则返回 `null` / 未就绪
- 返回的本地地址在 SDK 生命周期内**保持不变**，可缓存到业务层重复使用

## 下一步

业务请求接入完成后，启用 EdgeDoH 防 DNS 劫持 → [启用 EdgeDoH](enable-edgedoh.md)。
