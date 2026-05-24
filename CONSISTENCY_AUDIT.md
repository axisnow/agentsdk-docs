# 白皮书 vs 现有文档 一致性审查报告

> 日期：2026-05-19
> Ground truth：`docs/introduction/whitepaper.md`
> 审查对象：除白皮书外的所有 27 篇 md 文件
> 审查方法：按白皮书定义为基准，逐项核对其他文档的相关描述

---

## 一、重大不一致（P0 — 影响心智模型）

### 🔴 1.1 "控制面" 术语含义冲突（最严重）

| 来源 | "控制面" 指什么 |
|---|---|
| **白皮书** | "**AxisNow Cloud（控制面）**：策略编排、配置分发与全局观测中心。**不直接参与业务流量转发**" |
| **glossary.md** | `AxisNow Cloud`= 中心化的策略编排平台；`控制面节点`= **客户自部署的边缘节点**，运行 Edge DoH |
| **architecture.md §1** | 把 `控制面节点（Edge DoH）` 和 `AxisNow Cloud` 并列为两个独立组件 |
| **enable-edgedoh.md** | "EdgeDoH 是**控制面节点**提供的 DNS-over-HTTPS 解析服务" |

**问题**：同一个词 "控制面" 在白皮书里指云端（Cloud），在 SDK 文档里指边缘节点（≈ 白皮书的 Private Edge）。读者切换文档时心智模型完全错乱。

**根因**：SDK 文档诞生于白皮书之前，自创了"控制面节点 / 数据面节点"二元划分。白皮书后来用了更精确的"控制面 = Cloud / Multi-Edge = Private+Public Edge"。

**建议术语映射**：

| SDK 文档现用 | 白皮书规范 | 含义 |
|---|---|---|
| `控制面节点` | `Private Edge` | 同时运行 Edge DoH（调度面）+ 数据面 的自建边缘节点 |
| `数据面节点` | `Data Plane`（含 Private + Public） | 业务流量转发层（无论 Private 还是 Public） |
| `AxisNow Cloud` | `AxisNow Cloud（控制面）` | 云端编排与可观测中心 |

---

### 🔴 1.2 Private Edge / Public Edge / Multi-Edge 概念在 SDK 文档中完全缺失

**白皮书定义**：

- **Multi-Edge 网络** = `Private Edge` + `Public Edge`
- **Private Edge**（AxisNow 自建）：**双重职责** — 调度面（运行 Edge DoH）+ 数据面（TLS 加密链路 / WAF / Bot 防护）
- **Public Edge**（第三方 CDN，如 Cloudflare）：**仅数据面** — 由 Private Edge 上的 Edge DoH 决定调度，通过 JWT Token 鉴权

**现状**：除 whitepaper.md 外，`Private Edge` / `Public Edge` / `Multi-Edge` 三个词**零出现**。SDK 文档把所有边缘节点笼统称为"数据面节点"（注：可在 AxisNow / 第三方 / 自部署之间混部）。

**后果**：读完 architecture.md 的开发者不会知道 AxisNow 自建边缘和第三方 CDN 有本质不同（前者能跑 Edge DoH，后者不能；前者用 Secure Proxy，后者用 Standard Proxy）。

---

### 🔴 1.3 SecureProxy 定义把白皮书的两类模式揉成一个

| 来源 | 定义 |
|---|---|
| **白皮书 §请求链路 阶段二** | **两个并列模式**：<br>① **Secure Proxy 模式** — 与 **Private Edge** 建立加密链路<br>② **Standard Proxy 模式** — 向 **Public Edge** 注入 JWT Token |
| **glossary.md** | **SecureProxy** = SDK 端侧的"整体安全代理能力"，包含两类防护：加密通信隧道 + 基于可信设备环境的访问控制 |

**问题**：glossary 的 "SecureProxy" 是一个上位概念，把白皮书的 Secure Proxy（Private 模式）和 Standard Proxy（Public 模式）合二为一。读者无法区分两种端到端接入方式。

**衍生问题**：现有文档的 prerequisites / quickstart 都说"业务域名走 SecureProxy"，但实际上根据目标节点是 Private 还是 Public Edge，走的是不同模式。

---

## 二、中度不一致（P1 — 术语定义偏差）

### 🟠 2.1 EIP 是静态还是动态？

| 来源 | EIP 性质 |
|---|---|
| **白皮书** | 三层 EIP：<br>① **预置 EIP** — App SDK 内置兜底地址<br>② **AED 域名寻址** — 通过公共 DoH 解析<br>③ **策略分配 EIP** — **AED 定期下发的最优 EIP（动态、上下文相关）** |
| **glossary.md** | "EIP = **Elastic IP，控制面节点的固定入口 IP**。`edgeAddresses` 中以纯 IP 形式预埋的部分作为冷启动入口与最后兜底" |

**问题**：glossary 把 EIP 说成"固定入口 IP"，但白皮书强调 EIP 是动态分配的最优地址。glossary 描述的只是其中一层（预置 EIP）。

**修复建议**：glossary 中的 EIP 词条应当：
- 说明 EIP 是 **Edge DoH 服务的对外入口**（不是"控制面节点的固定 IP"）
- 说明 SDK 持有的 EIP 可能是 ①预置、②兜底域名解析得到、③ AED 策略下发 三类之一

---

### 🟠 2.2 拨测责任归属

| 来源 | 谁拨测谁 |
|---|---|
| **白皮书** | "**AxisNow Cloud** 还对 Data Plane 进行持续监控... **AxisNow Cloud 持续对各 Data Plane 节点进行拨测和健康检查**" |
| **glossary.md** | "**拨测** = **控制面节点对数据面节点**的健康度探测" |

**问题**：拨测的发起方在两份文档里不同。白皮书说是 Cloud 拨测；glossary 说是边缘节点拨测。

**注**：如果 glossary 的 "控制面节点" 严格映射到 Private Edge，那这条仍然不对——白皮书明确说是 Cloud 拨测。

---

### 🟠 2.3 调度域名 vs 业务域名 — 重要概念在 SDK 文档中完全缺失

**白皮书**：用整个 §路由机制 章节区分两类域名：

| 类型 | 例子 | 作用 |
|---|---|---|
| **调度域名** | `doh.acme.com` | 让 SDK 找到 Edge DoH。**先于业务请求被解析**，是系统启动前提，对极致可达性有要求 |
| **业务域名** | `api.app.com` | 应用实际访问的目标域名，依赖 Edge DoH 的上下文感知调度 |

**SDK 文档现状**：从未明确区分这两类域名。`edgeAddresses` 在 init-parameters.md 里被描述为 "Edge 节点的访问入口列表"——但实际上它放的就是**调度域名**（+ 预置 EIP）。`edgedoh_resolve_domains` 是**业务域名**白名单。

**后果**：开发者看到 `edgeAddresses` 和 `edgedoh_resolve_domains` 两个参数，但不知道它们属于两个完全不同的语义类别——一个是"找 Edge DoH 的入口"，另一个是"哪些业务域名走 Edge DoH 解析"。

---

### 🟠 2.4 白皮书"设计理念"与 overview"核心能力"无对应

三份文档给出三种归纳：

| 来源 | 归纳框架 |
|---|---|
| **白皮书 § 设计理念** | 4 条：① 客户端即第一道防线 ② 按信任开放 ③ 端到端可观测 ④ 多云边缘可灵活组合 |
| **白皮书 § 核心能力矩阵** | 5 维度：域名解析 / 应用安全加速 / 网络安全加速 / 业务安全 / 体验监控 |
| **overview.md § 核心能力** | 4 项：Context-aware Routing / Multi-Edge Proxy / 链路加密+流量签名 / 设备资源信任校验 |

**问题**：overview 的 4 项核心能力既不是设计理念的子集，也不严格对应能力矩阵的 5 个维度。同一站点内三种归纳并存，读者无法建立一致的产品认知。

---

## 三、轻度不一致（P2 — 表面或术语层面）

### 🟡 3.1 "资源"在白皮书里不是一个术语

**SDK 文档**：

- prerequisites.md：「控制台添加**资源**配置（可选）」「未包含在控制台**资源**中的域名，不会通过 SecureProxy 访问」
- glossary.md：「**资源** | 控制台为业务域名定义的代理对象，关联源站、路由规则、访问策略」

**白皮书**：从未使用 "资源" 作为术语。对应概念用"域名"、"源站"、"路由策略"、"安全策略"等具体词描述。

**问题**：读者从白皮书过来，找不到"资源"对应的概念；读 SDK 文档时不知道"资源"对应白皮书的哪个抽象。

---

### 🟡 3.2 客户端接入步骤数 — 2 步 vs 4 步

| 来源 | 步骤 |
|---|---|
| **白皮书 § 客户端集成** | "SDK 集成只需 **2 步**"：① 初始化 SDK ② 将网络请求接入 SDK（推荐 axhttp） |
| **guides/quickstart.md** | "**4 个步骤**"：① 初始化 ② 接入网络库 ③ 启用 EdgeDoH ④ 验证 |

**说明**：两者都对，但 framing 不同。白皮书的 2 步是"最少必需"，quickstart 的 4 步包含了"配置最常见场景 EdgeDoH"和"验证生效"两个附加步骤。

**建议**：在 quickstart.md 加一句"白皮书把这个流程压缩成最少必需的 2 步（init + 接入网络）"，让两份文档的关系明确。

---

### 🟡 3.3 业务安全细节描述深度差异

| 来源 | "设备/业务安全" 描述 |
|---|---|
| **白皮书** | 具体列举：**App Attestation**（应用完整性校验）/ **API 密钥保护** / **Certificate Pinning**（证书绑定）/ **设备环境检测**（越狱/Root/模拟器/调试/Hook 框架） |
| **overview.md** | 笼统：「结合**设备指纹、网络环境、风险评级**，对终端请求做可信判定」 |

**问题**：白皮书宣传了具体的能力清单；SDK 文档却笼统带过。开发者读 overview 不知道 SDK 实际能做 App Attestation / Cert Pinning 这些事情。

---

### 🟡 3.4 产品名混乱（3 种叫法）

| 出现位置 | 叫法 |
|---|---|
| 白皮书 H1 标题 | **AxisNow 移动/原生应用增强套件** 白皮书 |
| 白皮书 § 什么是 | "AxisNow 移动/原生应用增强套件" / "AxisNow SDK" / "AxisNow" |
| 其他 md 全文 | **AxisNow SDK** |
| mkdocs site_name | "AxisNow SDK 开发者文档" |
| 白皮书 Kotlin 示例（L419） | `AgentSDK.init` / `AgentConfig.Builder`（代码字面量） |

**建议**：冻结产品名为 **AxisNow SDK**；白皮书 H1 同步改名；代码示例中的 `AgentSDK` / `AgentConfig` 等待 SDK 包实际 API 命名确定后再批量替换。

---

### 🟡 3.5 SDK 与 Cloud 的通信路径描述差异

| 来源 | 描述 |
|---|---|
| **白皮书 § SDK Client** | "SDK 通过 **Edge DoH** 获取控制面下发的配置与策略，而非直接连接控制面" |
| **glossary.md / init-parameters.md** | "SDK 不与 Cloud 直连，所有交互经**控制面节点**中转" |

**说明**：内核语义一致——SDK 不直连 Cloud，中间走 Edge DoH（= 运行在 Private Edge 上的服务）。但 SDK 文档用"控制面节点"这个词，模糊了"中间节点是 Edge DoH 服务，本质上还是边缘节点"的事实。

---

## 四、白皮书覆盖但 SDK 文档未提及的内容

这些**不算"不一致"**，但属于"白皮书宣传的能力 / 设计点，SDK 文档没有体现"——读完白皮书来 SDK 文档找细节会扑空。

| 白皮书内容 | SDK 文档现状 | 建议位置 |
|---|---|---|
| **基于上下文的节点隔离**（高风险设备走不同节点池） | ❌ 未提 | `introduction/architecture.md` 新增章节 |
| **多边缘协同 4 策略**（地域 / 风险 / 灾备 / 成本） | ❌ 未提 | `introduction/architecture.md` 或 `guides/dns-routing.md` |
| **动态接入与故障切换 3 层**（Data Plane / Edge DoH / Edge DoH 寻址） | ⚠️ glossary 提到"兜底"但未系统说明 | `introduction/architecture.md §5` 扩充 |
| **去中心化机制**（接入点发现 / DNS / 数据面 / 客户端决策 4 层独立） | ❌ 未提 | `introduction/architecture.md` 或新章节 |
| **App Attestation / Cert Pinning / 设备环境检测** | ⚠️ overview 只笼统提"设备资源信任校验" | `introduction/overview.md` 扩充 |
| **5 大能力矩阵** | ⚠️ 用其他归纳替代了 | 与 overview "核心能力" 协调 |

---

## 五、不一致总览表

| # | 主题 | 严重度 | 涉及文件 |
|---|---|---|---|
| 1.1 | "控制面" 概念冲突（Cloud vs 节点） | 🔴 P0 | architecture, glossary, enable-edgedoh, system-architecture.mmd |
| 1.2 | Private/Public/Multi-Edge 缺失 | 🔴 P0 | architecture, glossary, 各 guides |
| 1.3 | SecureProxy 定义差异 | 🔴 P0 | glossary, prerequisites, quickstart |
| 2.1 | EIP 静态/动态 | 🟠 P1 | glossary, init-parameters |
| 2.2 | 拨测责任 | 🟠 P1 | glossary |
| 2.3 | 调度/业务域名缺失 | 🟠 P1 | architecture, init-parameters, dns-routing, enable-edgedoh |
| 2.4 | 设计理念归纳错位 | 🟠 P1 | overview |
| 3.1 | "资源" 不在白皮书 | 🟡 P2 | prerequisites, glossary |
| 3.2 | 步骤数 2 vs 4 | 🟡 P2 | quickstart |
| 3.3 | 业务安全细节 | 🟡 P2 | overview |
| 3.4 | 产品名 3 种 | 🟡 P2 | whitepaper, mkdocs, 多处 |
| 3.5 | SDK↔Cloud 通信路径 | 🟡 P2 | glossary, init-parameters |

---

## 六、修复优先级建议

### 第一波 — 术语对齐（1 天）
统一术语表（建议直接在 glossary.md 中新增/修正）：

1. 引入 **Private Edge** / **Public Edge** / **Multi-Edge** / **Data Plane** 4 个新术语
2. **重写"控制面节点"词条**：明确它是 Private Edge 的别称，**不要**与 AxisNow Cloud (白皮书的"控制面") 混用
3. **拆开 SecureProxy 词条**为 Secure Proxy / Standard Proxy 两个，对应 Private/Public Edge
4. **修正 EIP 词条**：说明三层动态结构
5. **修正拨测词条**：明确发起方是 AxisNow Cloud
6. **删除/重定义 "资源" 词条**：要么改用白皮书的"域名 + 源站 + 路由策略"组合描述，要么在词条里说明"资源是 SDK 文档对'域名 + 源站 + 策略' 的统称"

### 第二波 — 关键章节扩充（2-3 天）

7. `introduction/architecture.md` 大改：
   - §1 整体工作流程：使用白皮书的 4 组件模型（SDK / Edge DoH / Multi-Edge / Cloud）
   - 新增"调度域名 vs 业务域名" §
   - 新增"动态接入与故障切换" §
   - 新增"多边缘协同 / 节点隔离 / 去中心化" 简介或链接
8. `introduction/overview.md` 调整：
   - 把"核心能力"重命名/改写以对齐白皮书的 4 设计理念或 5 能力维度
   - 业务安全描述补充具体能力清单（App Attestation / Cert Pinning / 设备检测）

### 第三波 — 接入流程文档微调（0.5 天）

9. `guides/quickstart.md`：加一句脚注，说明 4 步流程对应白皮书的 2 步压缩
10. `guides/prerequisites.md`：把"控制台添加资源配置"和"DoH 路由规则"用白皮书一致的词描述
11. `guides/initialize.md` / `dns-routing.md`：在描述 `edgeAddresses` 时明确它是"调度域名+EIP"；描述 `edgedoh_resolve_domains` 时明确它是"业务域名白名单"

### 第四波 — 产品名收敛（0.5 天）

12. 白皮书 H1 改 "AxisNow SDK 白皮书"
13. 白皮书内"增强套件"一律改 "SDK"
14. 代码示例中的 `AgentSDK` / `AgentConfig` 暂保留，标 `TODO`：等真实 API 命名确定后批量替换

---

## 七、特别说明

**关于 architecture.md 图片**：`docs/assets/images/overall-workflow.png` 和 `system-architecture.png` 这两张图按 SDK 文档的旧模型（控制面节点 / 数据面节点）绘制，与白皮书的图（SDK / Edge DoH / Multi-Edge / Cloud）不一致。第二波中修改 architecture.md 时需重新生成对应图。

**关于自动测试**：建议在 CI 中加一条 lint：禁止 docs/ 内除 internal/ 外的文件使用"控制面节点"（除非显式标注为"= Private Edge"）。这能防止术语回归。
