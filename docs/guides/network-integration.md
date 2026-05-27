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

## 调用约束

- 所有代理接口在 `initialize` 成功（返回 `0`）后才可调用，否则返回 `null` / 未就绪
- 返回的本地地址在 SDK 生命周期内**保持不变**，可缓存到业务层重复使用

## 下一步

业务请求接入完成后，启用 EdgeDoH 防 DNS 劫持 → [启用 EdgeDoH](enable-edgedoh.md)。
