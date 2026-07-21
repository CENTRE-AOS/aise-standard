# CENTRE Gateway Runtime Architecture Constitution

> 版本: 1.0.0frozen
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: aos-runtime v3.2.0
> 前置: CENTRE Protocol Constitution (aise-standard)

---

## Chapter 0 — Prime Directive

### 0.1 优先级法则

CENTRE Gateway Runtime 永远服从以下优先级：

```
CENTRE Architecture Principles
    >
AISE Protocol
    >
Identity Validity
    >
Admission Policy
    >
Runtime Policy
    >
Agent Request
```

Agent 请求是最低优先级。CENTRE Architecture Principles 是最高原则——即使 AISE Protocol 未来演化，也不得违反。

### 0.2 优先级解释

```
Identity Validity:  你是谁，身份是否合法
Admission Policy:   你是否被允许进入此 Project
```

身份合法不代表准入通过。类似：身份证不等于门禁权限。

### 0.3 不可违背原则

1. **架构至上**：CENTRE Architecture Principles 是最高原则，AISE Protocol 和所有 Runtime 必须服从
2. **协议至上**：AISE Protocol 是唯一工程协议，Runtime 只执行，不定义
3. **身份优先**：Project 是独立生命实体，Runtime 不拥有、不管理 Project
4. **准入强制**：任何 Agent 进入 Project 必须通过 Admission 验证，无例外
5. **接口不变**：v2.0 冻结的接口签名，后续版本不得变更

---

## Chapter 1 — Identity

### 1.1 我是什么

CENTRE Gateway Runtime 是 AISE Protocol 的**参考执行实现**（Reference Runtime）。

```
CENTRE Architecture   ← 世界模型 / 架构体系
    │
    ├── AISE Protocol  ← 唯一工程协议
    │
    └── Gateway Runtime ← 参考执行实现（本仓库）
```

### 1.2 Reference Runtime 兼容性原则

Reference Runtime 必须保持 AISE Protocol 完整兼容，但不代表所有 Runtime 必须采用相同实现。未来可以存在 Cloud Runtime、Workbench Runtime、Embedded Runtime 等，只要它们通过 AISE Protocol 兼容性验证即可。

### 1.3 物理身份

```
逻辑身份: CENTRE Gateway Runtime
物理仓库: aos-runtime（仓库名冻结，不随逻辑升级改名）
协议仓库: aise-standard（CENTRE Protocol 定义来源）
```

### 1.4 我不是什么

| 不是 | 原因 |
|------|------|
| 协议定义者 | 协议由 aise-standard 定义，Runtime 只执行 |
| 项目管理平台 | Project 是独立生命实体，拥有自己的 CENTRE Identity |
| Agent 调度器 | Agent 自主决策，Runtime 只验证准入，不管理生命周期 |
| CENTRE OS 本身 | Runtime 只是执行层，不是架构体系全部 |
| 唯一执行环境 | 未来可能存在其他 Reference Runtime |

---

## Chapter 2 — Universe Model

### 2.1 CENTRE 架构体系

```
CENTRE Architecture
    │
    ├── AISE Protocol（唯一工程协议，一号协议）
    │
    ├── Gateway Runtime（参考执行实现）
    │
    ├── Identity System（身份契约）
    │
    ├── Context System（上下文内核）
    │
    └── Presentation Layer（Workbench / Trae / CLI）
```

### 2.2 层级关系

```
HUMAN（价值定义者）
    │
CENTRE Architecture（世界模型）
    │
    ├── AISE Protocol（秩序规则）
    │
    ├── Gateway Runtime（规则执行）
    │
    └── Ecosystem（生态实例）
        │
        ├── Project（生命实体）
        ├── Agent（持证执行者）
        ├── Human（价值持有者）
        ├── Tool（能力扩展）
        └── Runtime（执行环境）
```

### 2.3 Agent 准入链路（唯一路径）

```
任何 Agent
    │
    ├── 1. 检测 .project/centre.protocol.json
    ├── 2. 验证协议清单（hash + version + schema）
    ├── 3. 验证 Runtime 版本兼容性
    ├── 4. 通过 → 加载 AISE Protocol → 执行
    └── 5. 不通过 → 拒绝，输出明确原因

无例外。无绕过。无临时豁免。
```

---

## Chapter 3 — Entity Model

### 3.1 生态实体

CENTRE Ecosystem 由五类实体组成：

| 实体 | 身份标识 | 职责 | Identity Provider |
|------|---------|------|-------------------|
| Project | `.project/centre.identity.json` | 持有代码、记忆、决策 | Human / Organization |
| Agent | `.agent/centre.identity.json` | 执行任务 | Human / Agent Provider |
| Human | `.human/centre.identity.json` | 定义价值与目标 | Identity Provider |
| Tool | `.tool/centre.identity.json` | 提供能力 | Human / Provider |
| Runtime | `CENTRE_HOME/state/identity.json` | 执行协议 | Runtime Provider |

> Runtime 自身也是生态实体，拥有独立身份。未来 Cloud Runtime、Workbench Runtime、Embedded Runtime 各自拥有独立身份。Runtime 不创建任何其他实体的 Identity，只验证 Identity。

### 3.2 Identity Provider

v2.0 使用 Local Identity Provider（实体自行管理身份文件）。接口预留：

```
v2.0:   Local Identity Provider（实体自行管理）
v3.0+:  CENTRE Identity Authority（统一签发）
```

### 3.3 Project 优先原则

Project 是 CENTRE Ecosystem 的第一公民。所有其他实体服务于 Project：

```
Agent    → 服务于 Project
Tool     → 服务于 Project
Human    → 定义 Project 价值
Runtime  → 为 Project 提供准入验证
```

### 3.4 身份契约（不存储）

Runtime 定义身份契约，不存储身份数据：

```
身份契约 = 类型 + 指纹 + 元数据
身份存储 = 由实体自行管理
身份证明 = v2.0 Local Provider / v3.0+ CENTRE Identity Authority
```

---

## Chapter 4 — Runtime Boundary

### 4.1 我的职责

```
├── 加载并执行 AISE Protocol
├── 验证 Protocol Manifest（准入）
├── 管理 CENTRE_HOME 环境
├── 提供身份契约（IIdentity）
├── 提供准入协议（IAdmission）
├── 提供上下文内核（IContext）
├── 提供能力契约（ICapability）
├── 提供适配器接口（IAdapter）
└── 提供事件契约（IEvent）
```

### 4.2 不是我的职责

```
├── 定义协议规则 → 属于 aise-standard
├── 存储项目记忆 → 属于 Workbench Memory System
├── 管理 Agent 生命周期 → Agent 自主管理
├── 签发加密证书 → 属于未来 Certificate Authority
├── 运行事件总线 → 属于未来 CENTRE OS Kernel
├── 调度 Agent → Agent 自主决策
├── 拥有项目 → Project 是独立生命实体
└── 解析项目内容 → Project 自己负责，不属于 Runtime
```

### 4.3 明确边界：Agent 生命周期

Runtime 不管理 Agent 生命周期。Runtime 的职责止于准入验证：

```
Runtime 做:
    ├── 验证 Agent 身份
    ├── 提供执行环境
    └── 记录事件

Runtime 不做:
    ├── 启动 Agent
    ├── 关闭 Agent
    ├── 调度 Agent
    └── 管理 Agent 状态
```

### 4.4 明确边界：Project 内容

Runtime 不解析 Project 内容。Runtime 不读取代码、不分析结构、不评估质量。这是防止 Runtime 演变为 IDE 的关键边界。

---

## Chapter 5 — Frozen Interfaces

以下 6 个接口在 v2.0.0frozen 中冻结，后续版本不得变更签名。

### 5.1 IIdentity

```
优先级: 入口（准入的前提）
```

```powershell
# 身份契约 — 定义实体身份，不存储
function New-Identity {
    param(
        [ValidateSet("project", "agent", "human", "tool", "runtime")]
        [string]$Type,
        [hashtable]$Metadata
    )
    # 返回: @{ id; type; fingerprint; created_at }
}

function Test-Identity {
    param([hashtable]$Identity)
    # 返回: bool
}
```

### 5.2 IAdmission

```
优先级: 核心（Agent 进入项目的唯一闸门）
```

```powershell
# 准入协议 — 验证 Protocol Manifest，执行准入判定
function Invoke-Admission {
    param([string]$ProjectPath)
    # 返回: @{ allowed: bool; reason: string; protocol: string }
    # 验证流程:
    #   1. 检测 .project/centre.protocol.json
    #   2. 验证 fingerprint (hash)
    #   3. 验证 protocol_version 兼容性
    #   4. 验证 runtime_min_version 兼容性
}
```

### 5.3 IContext

```powershell
# 上下文内核 — 维护引用，不存储记忆
function Get-ContextSnapshot {
    param([string]$ProjectPath)
    # 返回: @{ project_context; session_context; agent_context; external_reference }
}

function Resolve-ContextReference {
    param([string]$ReferencePath)
    # 解析上下文引用路径
}

function Test-ContextIntegrity {
    param([hashtable]$Snapshot)
    # 返回: bool
}
```

> `external_reference` 指向外部知识系统（Workbench Memory / Knowledge Graph / Project Intelligence），Runtime 只维护引用，不存储内容。`Merge-Context` 和 `Restore-Context` 属于 Workbench Handoff 层，不在此接口中。

### 5.4 ICapability

```powershell
# 能力契约 — 描述 Agent 能力，不定义适配器
function Get-AgentCapability {
    # 返回: @{ language; tools; model; limits }
}

function Test-CapabilityMatch {
    param([hashtable]$Required, [hashtable]$Available)
    # 返回: bool
}
```

### 5.5 IAdapter

```powershell
# 适配器接口 — 表现层注入，不是协议层
function Test-AgentPresence {
    # 检测当前 Agent 环境是否存在
    # 返回: bool
}

function Install-EntryPoint {
    param([string]$ProjectPath)
    # 注入 AISE Protocol 入口规范到 Agent 配置
    # 返回: bool
}

function Test-Installation {
    # 验证注入是否生效
    # 返回: bool
}
```

### 5.6 IEvent

```powershell
# 事件契约 — 定义事件结构，不实现总线
function New-Event {
    param(
        [string]$Type,        # agent.enter, agent.exit, action.completed, action.failed
        [string]$Source,      # trae, claude, cli
        [hashtable]$Payload
    )
    # 返回: @{ id; type; source; timestamp; payload }
}
```

### 5.7 接口链路

```
IIdentity → IAdmission → IContext → ICapability → IAdapter → IEvent
  (身份)      (准入)      (上下文)     (能力)       (适配)     (事件)
```

没有身份，不能准入。准入通过后，才能加载上下文。上下文就绪后，才能匹配能力。能力匹配后，才能注入适配器。所有动作记录为事件。

---

## Chapter 6 — Protocol Manifest

### 6.1 定位

v2.0 实现**协议清单（Protocol Manifest）**，不实现加密证书体系。

```
v2.0:   Protocol Manifest (hash + version + schema)
v3.0+:  CENTRE Certificate (RSA/ECC 签名)
v4.0+:  Encrypted Identity Token (动态加密)
```

### 6.2 文件结构

```
.project/
├── centre.protocol.json    ← v2.0 实现（协议清单）
├── centre.identity.json    ← 未来实现（项目身份）
└── centre.context.json     ← 未来实现（上下文配置）
```

三者分离的原因：Identity 生命周期长，Protocol 随版本变化，Context 动态变化。不可合并。

### 6.3 centre.protocol.json Schema（冻结字段）

```json
{
  "protocol_id": "AISE",
  "protocol_version": "2.6.0",
  "runtime_min_version": "2.0.0",
  "schema_version": "1.0",
  "fingerprint": "sha256:xxx",
  "created_at": "2026-07-17T00:00:00Z"
}
```

字段语义：

| 字段 | 含义 | 示例 |
|------|------|------|
| `protocol_id` | 协议标识 | AISE |
| `protocol_version` | 协议版本 | 2.6.0 |
| `runtime_min_version` | 最低 Runtime 版本 | 2.0.0 |
| `schema_version` | Manifest Schema 版本 | 1.0 |
| `fingerprint` | 完整性校验 | sha256:xxx |

> `version` 不允许同时承担三个含义。`protocol_id`、`protocol_version`、`schema_version` 职责分离。

### 6.4 验证逻辑

```
1. 检测 .project/centre.protocol.json
2. 校验 protocol_id == "AISE"（当前唯一协议）
3. 校验 protocol_version 与已安装协议匹配
4. 校验 runtime_version >= runtime_min_version
5. 校验 schema_version 兼容
6. 校验 fingerprint 完整性 (SHA256 hash)
7. 全部通过 → 准入
```

---

## Chapter 7 — CENTRE_HOME Contract

### 7.1 目录结构

```
CENTRE_HOME/                    # 默认: ~/.aos
├── runtime/                    # Runtime 可执行文件
│   ├── core/
│   └── cli/
├── protocols/                  # 协议 Artifact（只读，不可变）
│   └── aise/
│       ├── current → versions/x.x.x/
│       └── versions/
├── extensions/                 # 扩展（适配器、能力提供者、插件）
│   └── adapters/
├── registry/                   # 全局注册表
├── context/                    # 上下文引用
├── logs/                       # 运行日志
└── state/                      # 环境状态（含 Runtime 自身身份）
```

### 7.2 环境变量

```
CENTRE_HOME  → 默认 ~/.aos
AOS_HOME     → 向后兼容别名，指向 CENTRE_HOME
```

### 7.3 不在 CENTRE_HOME 中的内容

- **宪法文件**：属于 Protocol Artifact，随协议一起安装，不单独存放
- **缓存**：临时文件，不进入架构定义
- **项目文件**：项目属于 Project，不属于 Runtime 环境
- **适配器实现**：属于 `extensions/`，不属于 `runtime/`

---

## Chapter 8 — Repository Convention

### 8.1 物理仓库（冻结，不 rename）

| 仓库名 | 逻辑身份 | 职责 |
|--------|---------|------|
| `aise-standard` | CENTRE Protocol Specification | 定义协议，不执行 |
| `aos-runtime` | CENTRE Gateway Runtime | 执行协议，不定义 |

### 8.2 Repository Sovereignty

```
aise-standard
拥有协议定义权

aos-runtime
拥有执行实现权

Runtime 不得修改 Protocol
Protocol 不依赖 Runtime 实现
```

这是防止 Runtime 反向定义协议的根本约束。

### 8.3 版本号

```
仓库版本:   2.0.0frozen    (VERSION 文件)
协议版本:   1.0             (AISE Protocol 版本，.agent-entry.json)
Runtime:    2.0.0frozen    (与仓库版本一致)
```

### 8.4 Git 分支规则

```
main:    只读，仅接受 frozen tag merge
develop: 开发分支
tag:     v2.0.0frozen (不可变，不可 force-move)
```

---

## Chapter 9 — Non-Scope

v2.0.0frozen 明确排除以下能力：

| 排除项 | 原因 | 接口是否预留 |
|--------|------|-------------|
| RSA/ECC 加密签名 | SHA256 当前够用 | 是 — 通过 fingerprint 字段 |
| 证书撤销机制 | 无签发中心 | 否 |
| 多协议支持（非 AISE） | AISE 是唯一协议 | 否 |
| 分布式事件总线 | 单机够用 | 是 — IEvent |
| 身份存储后端 | 当前 JSON 够用 | 是 — IIdentity |
| 记忆存储 | 属于 Workbench | 是 — IContext.external_reference |
| Agent 生命周期管理 | Agent 自主管理 | 否 |
| Agent 调度 | Agent 自主决策 | 否 |
| Project 内容解析 | Project 自己负责 | 否 |
| Gateway 网络层 | 非基础设施 | 否 |
| Agent 通信协议 | 表现层 | 否 |
| 自建协议分发系统 | GitHub 当前够用 | 是 — install URL 可替换 |
| MCP 整合 | 生态阶段 | 否 |
| 多 Agent 适配器实现 | 仅实现 Trae | 是 — IAdapter |

---

## Chapter 10 — Evolution Path

### 10.1 版本路线

```
v2.0.0frozen  ← 当前目标
  │  基座冻结：6 个接口 + Protocol Manifest + CENTRE_HOME + AISE Admission
  │
v2.1.0+  (Stabilization)
  │  多平台测试（Windows/Linux/macOS）
  │
v2.5.0+  (Capability)
  │  加密证书体系（RSA/ECC）
  │  多 Agent 适配器
  │
v3.0.0+  (Ecosystem)
  │  自建协议分发中心
  │  Gateway 网络化
  │  分布式 Identity
  │  事件总线
```

### 10.2 基座不变原则

```
v2.0.0frozen 冻结的 6 个接口签名，v3.0, v4.0 不得变更。
只能新增接口，不能修改已有接口。
```

---

## 附录 A — 与 AISE Protocol Constitution 的关系

| 文档 | 位置 | 职责 |
|------|------|------|
| AISE Protocol Constitution | `aise-standard/constitution/CONSTITUTION.md` | 定义 AISE 工程协议规则 |
| CENTRE Gateway Runtime Constitution | `aos-runtime/constitution/CONSTITUTION.md` | 定义 Runtime 身份与边界 |

两者关系：

```
AISE Protocol Constitution  → 定义"Agent 如何工作"
CENTRE Runtime Constitution → 定义"Runtime 如何执行协议"
```

---

## 附录 B — 修订记录

| 版本 | 日期 | 修订内容 |
|------|------|---------|
| 1.0-draft | 2026-07-17 | 初稿 |
| 1.0-rc | 2026-07-17 | 8 处修正 |
| 1.0.0frozen | 2026-07-17 | 3 处最终修正：Prime Directive 优先级增加 Identity Validity/Admission Policy 区分；Entity Model 增加 Runtime 实体；Protocol Manifest 字段标准化（protocol_id / protocol_version / schema_version 分离） |