# 错误码

AgentSDK 所有 API 返回 `0` 表示成功，**负数表示失败**。错误码按子系统分段：

| 段 | 范围 | 用途 |
|---|---|---|
| 通用 | `-1` ～ `-99` | 跨模块基础错误 |
| 初始化与配置 | `-101` ～ `-199` | SDK 初始化、AccessKey、Edge 地址等 |
| DNS | `-201` ～ `-299` | DNS 解析失败、超时、上游不可用 |
| Proxy | `-301` ～ `-399` | 本地代理监听、端口、文件描述符 |

如需结合**排障流程**定位问题，参见 [排障](troubleshooting.md)。

## 通用

| 错误码 | 名称 | 含义 | 常见原因 | 解决方案 |
|--------|------|------|---------|---------|
| `0` | OK | 成功 | — | — |
| `-1` | Unknown | 未知错误 | 未预期的异常（非典型错误，多为上游漏 wrap） | 查看 SDK 日志并联系技术支持 |
| `-2` | InvalidArgument | 参数非法 | host 为空、port 超出范围、格式错误 | 检查传入参数的合法性 |
| `-50` | Internal | 内部错误 | 设备信息采集失败、存储初始化失败、JSON marshal 失败 | 检查设备权限和存储可用性 |

## 初始化与配置

| 错误码 | 名称 | 含义 | 常见原因 | 解决方案 |
|--------|------|------|---------|---------|
| `-101` | NotInitialized | SDK 未初始化 | 在初始化完成前调用了其他 API | 确保 `initialize` 成功返回后再调用 |
| `-102` | AlreadyInitialized | 重复初始化 | 多次调用 `initialize` | 使用单例模式确保只调用一次 |
| `-111` | InvalidConfig | 配置 JSON 非法 | JSON 格式错误、字段类型不匹配 | 检查 JSON 是否合法、字段是否符合规范 |
| `-112` | AccessKeyIDMissing | `access_key_id` 为空 | 未填或填了空字符串 | 在配置中提供有效的 AccessKey ID |
| `-113` | AccessKeySecretMissing | `access_key_secret` 为空 | 未填或填了空字符串 | 在配置中提供有效的 AccessKey Secret |
| `-114` | AccessKeyInvalid | AccessKey 内容非法 | ID 非 UUID 格式、Secret 解码或解密失败、解出空 tenant/deployment | 重新从控制台复制完整凭证 |
| `-115` | EdgeAddressEmpty | `edge_addresses` 为空 | 未配置或数组里全是空字符串 | 至少配置一个有效的 Edge 节点地址 |

## DNS

| 错误码 | 名称 | 含义 | 常见原因 | 解决方案 |
|--------|------|------|---------|---------|
| `-201` | DNSNXDomain | 域名不存在（预留） | NXDOMAIN（当前走成功路径返回空 IPs，预留供未来使用） | 检查域名拼写 |
| `-211` | DNSTimeout | 解析超时 | 网络抖动 / 上游 DNS 慢 | 重试；检查网络与超时配置 |
| `-221` | DNSUpstreamUnavailable | 所有上游 DNS 失败 | 网络不可达、所有 race provider 都挂 | 检查设备网络、Edge 节点可达性 |
| `-231` | DNSInternal | 解析器内部错 | 无可用 engine、unknown strategy 等配置/状态错 | 检查 SDK 版本与 DNS 配置 |

## Proxy

| 错误码 | 名称 | 含义 | 常见原因 | 解决方案 |
|--------|------|------|---------|---------|
| `-301` | ProxyListenerUnavailable | 代理 listener 通用兜底 | 不可识别的 socket 系统错、iOS 重建失败 | 重启 SDK；上报技术支持 |
| `-311` | ProxyAddrInUse | 端口被占用 | EADDRINUSE，三层重试 + OS-assign 仍失败（极端端口竞争） | 重启进程；排查端口占用 |
| `-321` | ProxyFDExhausted | 文件描述符耗尽 | EMFILE/ENFILE，应用 FD 泄漏或系统限制低 | 排查 FD 泄漏；调高系统 ulimit |
| `-331` | ProxyPermissionDenied | 监听权限被拒 | EACCES/EPERM（罕见，通常是部署环境问题） | 检查应用是否被沙箱/安全策略限制 |

## 高频错误速查

如果你只关心最常见的几种，下面这一张表足够大多数场景：

| 错误码 | 含义 | 快速排查 |
|--------|------|---------|
| `-2` | 参数非法 | 检查 host 和 port |
| `-50` | 内部错误 | 检查存储 / Keystore / Keychain 权限 |
| `-101` | SDK 未初始化 | 确保先调用 initialize |
| `-102` | 重复初始化 | 只调用一次 initialize |
| `-111` | 配置 JSON 非法 | 检查 JSON 格式与字段类型 |
| `-112` ～ `-114` | AccessKey 凭证问题 | 重新从控制台复制完整凭证 |
| `-115` | EdgeAddresses 为空 | 至少配置一个 Edge 节点地址 |
| `-211` | DNS 超时 | 重试、检查网络 |
| `-221` | DNS 上游全失败 | 检查设备网络与 Edge 可达性 |
| `-301` ～ `-331` | 代理 listener 故障 | 见 [排障 §3.2](troubleshooting.md#32-代理连接问题) |
