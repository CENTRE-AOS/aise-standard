# RFC-0013: Runtime Boundary Enforcement

> RFC ID: RFC-0013
> Title: Runtime Boundary Enforcement — Runtime Authority, Validation Chain, State Ownership
> Version: 1.0.0
> Status: Frozen
> Type: Protocol Contract
> Date: 2026-07-21
> Author: CENTRE Protocol Authority
> Category: Constitutional
> Requires: RFC-0002 (Kernel Contract), RFC-0007.2 (Production Architecture), RFC-0012 (Federation Layer)
> Scope: aise-standard (Protocol Layer) — defines Runtime enforcement contract, NOT Runtime implementation
> Freeze Target: CENTRE Foundation Freeze v3.4

---

## 1. Abstract

定义 CENTRE Runtime 的强制执行边界。此 RFC 不是 Runtime 实现规范，而是 **Protocol 层对 Runtime 的契约约束**——Runtime 能做什么、不能做什么、如何验证传入请求、以及 State 的归属。

核心目标：证明 Foundation Contract 可以被执行。一个 Agent 可以在 CENTRE Contract 下被创建、约束、运行和验证。

---

## 2. Motivation

Foundation Freeze v3.3 冻结了 Federation Layer（Protocol 子领域）。下一步不是继续扩 Protocol，而是让 Runtime 消化 Protocol——证明从 Protocol Contract 到 Runtime Enforcement 的链路是完整的。

当前缺口：

| 已定义 | 未定义 |
|--------|--------|
| Protocol 规则 (RFC-0002) | Runtime 执行这些规则时的精确边界 |
| Runtime 组件 (Kernel 8) | 每个组件的 Authority 边界 |
| Federation Admission (FG-0~FG-4) | Runtime 如何消费 Federation Verified Entity |
| State 概念 (Context, Memory) | State 的精确所有权归属 |

---

## 3. Decision

### 3.1 Runtime Authority — 三层权限模型

Runtime 的操作权限分为三个层级：

```
CAN        — Runtime 有能力执行
ALLOWED    — Protocol 授权 Runtime 执行
FORBIDDEN  — Protocol 明确禁止 Runtime 执行
```

| 操作 | CAN | ALLOWED | FORBIDDEN |
|------|:--:|:------:|:---------:|
| Execute Protocol | ✅ | ✅ | — |
| Manage Lifecycle | ✅ | ✅ | — |
| Invoke Capability | ✅ | ✅ | — |
| Emit Events | ✅ | ✅ | — |
| Validate Contract | ✅ | ✅ | — |
| Route Events | ✅ | ✅ | — |
| Enforce Contract | ✅ | ✅ | — |
| **Modify Protocol Authority** | ❌ | ❌ | ✅ |
| **Redefine Identity** | ❌ | ❌ | ✅ |
| **Own Federation State** | ❌ | ❌ | ✅ |
| **Become Ecosystem Registry** | ❌ | ❌ | ✅ |
| **Own Project State** | ❌ | ❌ | ✅ |
| **Define Protocol Rules** | ❌ | ❌ | ✅ |
| **Manage Agent Behavior** | ❌ | ❌ | ✅ |

### 3.2 Runtime 不是

| 错误认知 | 正确归属 |
|---------|---------|
| Runtime 是 Protocol 定义者 | Protocol 定义规则，Runtime 执行规则 |
| Runtime 是生态注册中心 | Registry 是 Protocol 概念，Runtime 只缓存 |
| Runtime 是 Agent 管理器 | Agent 属于 Host 层，Runtime 只验证准入 |
| Runtime 是数据存储 | Runtime 不存储业务数据，Project 拥有自己的 State |
| Runtime 是中心服务器 | Runtime 是本地侧基础设施 |

---

## 4. Incoming Request Validation Chain

Runtime 接收任何请求时，必须执行以下验证链：

```
Incoming Request
        │
        ▼
[1] Protocol Validation
        │  验证请求是否符合 Protocol Contract
        │  Failure → REJECT (Protocol Violation)
        ▼
[2] Identity Verification
        │  验证请求方身份（Federation Verified Entity）
        │  Failure → REJECT (Identity Unknown)
        ▼
[3] Runtime Policy Check
        │  验证请求是否在 Runtime Authority 范围内
        │  Failure → REJECT (Out of Authority)
        ▼
[4] Execution
        │  执行请求
        │  产生 Event (success/failure)
        │  更新 Runtime State (非 Project State)
        ▼
Response + Event
```

### 4.1 Protocol Validation

验证依据：`aise-standard/protocol/` 中定义的 Contract Schema。

验证内容：
- 请求结构是否符合 Contract Schema
- 请求参数是否在合法范围内
- 请求是否引用了存在的 Contract 版本

### 4.2 Identity Verification

验证依据：Federation Admission Certificate（FG-0~FG-4 通过后的 `PROJECT_BOOTSTRAP_READY.md`）。

验证内容：
- 请求方是否持有有效的 Federation Certificate
- Certificate 中的 Authority Level 是否与请求操作匹配
- Certificate 是否未过期/未失效

### 4.3 Runtime Policy Check

验证依据：本 RFC §3.1 的 Runtime Authority 表。

验证内容：
- 请求的操作是否在 Runtime ALLOWED 列表内
- 请求的操作是否在 Runtime FORBIDDEN 列表内
- 请求是否试图修改 Protocol Authority / Identity / Federation State

### 4.4 Execution

通过以上三层验证后，Runtime 执行请求。执行结果以 Event 形式输出，不修改 Project State。

---

## 5. State Ownership

CENTRE 生态中 State 的精确归属：

| State 类型 | Owner | Stored In | Runtime Access |
|-----------|-------|-----------|:-------------:|
| **Protocol Definition** | Protocol Authority (A0) | `aise-standard/` | Read-Only |
| **Federation Contract** | Protocol Authority (A0) | `aise-standard/protocol/federation/` | Read-Only |
| **Runtime Execution State** | Runtime (A1) | Runtime memory | Read-Write |
| **Project State** | Project | `.project/` | Read (via Context API) |
| **Agent Memory** | Agent / Project | `.project/memory/` | Read (via Context API) |
| **Artifact** | Factory (A2) | Artifact Registry (A3) | Read-Only |
| **Certificate** | Federation | `PROJECT_BOOTSTRAP_READY.md` | Read-Only |

### 5.1 核心原则

```
Runtime owns execution state, NOT business state.
Runtime executes protocol, NOT owns protocol.
Runtime validates identity, NOT defines identity.
Runtime routes events, NOT owns events.
```

### 5.2 禁止的 State 操作

| 禁止操作 | 原因 |
|---------|------|
| Runtime 修改 Project State | Project 拥有自己的 State |
| Runtime 存储 Agent Memory | Memory 属于 Agent / Project |
| Runtime 缓存 Federation State | Federation 不拥有持久状态 |
| Runtime 修改 Protocol Definition | Protocol 属于 Protocol Authority |
| Runtime 创建 Global State Store | Runtime 不是数据存储 |

---

## 6. Runtime Lifecycle States

Runtime 自身有明确定义的生命周期，每个状态有明确的允许操作：

```
INIT → LOAD → READY → SERVE → STOP
  │                        │
  └──────── ERROR ←────────┘
```

| State | 允许操作 | 禁止操作 |
|-------|---------|---------|
| INIT | 初始化环境、加载配置 | 接受外部请求 |
| LOAD | 加载 Protocol、验证 Contract | 执行 Protocol |
| READY | 验证完成、等待启动 | 接受外部请求 |
| SERVE | 接受请求、执行 Protocol | 修改 Protocol |
| STOP | 清理资源、保存状态 | 接受新请求 |
| ERROR | 记录错误、等待恢复 | 正常执行 |

---

## 7. Runtime → Project Boundary

Runtime 与 Project 之间的交互必须通过 Contract 定义：

```
Runtime                     Project
   │                            │
   │  Context Request           │
   │ ─────────────────────────> │
   │                            │
   │  Context Snapshot          │
   │ <───────────────────────── │
   │                            │
   │  Event (execution result)  │
   │ ─────────────────────────> │
   │                            │
```

Runtime 不直接访问 Project 文件系统。所有交互通过 Context API。

---

## 8. Federation Integration

Runtime 消费 Federation 产出，但不参与 Federation 判断：

```
Federation Layer (Protocol)
        │
        │ produces:
        │   - Verified Entity Identity
        │   - Admission Certificate
        │   - Authority Binding
        │
        ▼
Runtime (Enforcement)
        │
        │ consumes:
        │   - Federation Verified Entity
        │   - Certificate (read-only)
        │
        │ does NOT:
        │   - Judge identity
        │   - Issue certificates
        │   - Manage federation state
        ▼
Execution
```

Runtime 接受 Federation 验证后的实体，不重复验证身份。Runtime 是"执行者"而不是"判断者"。

---

## 9. Impact Analysis

### 9.1 Protocol Layer

- 新增 Runtime Enforcement Contract（本 RFC）
- 不影响现有 Protocol Contracts

### 9.2 Runtime Layer

- 本 RFC 是 Protocol 对 Runtime 的契约约束
- Runtime 实现必须遵守 §3 的 Authority 边界
- Runtime 实现必须执行 §4 的验证链
- 不要求 Runtime 立即实现所有功能（v0.1 Minimal）

### 9.3 Foundation Freeze

本 RFC 触发 CENTRE Foundation Freeze v3.4，新增：
- Principle #9: Runtime Boundary Enforcement

### 9.4 后续阶段

| Phase | 内容 |
|-------|------|
| RFC-0013 | 本 RFC — Runtime Enforcement Contract |
| A1 Runtime Agent | Runtime 骨架实现，证明 Contract 可执行 |
| Runtime Closure | 验证 Runtime 实现与 Contract 一致性 |
| Runtime v0.1 | 最小可用 Runtime 实例 |

---

## 10. Relationship to Existing RFCs

| RFC | 内容 | 关系 |
|-----|------|------|
| RFC-0002 | Kernel Contract | 本 RFC 定义 Kernel 的 Authority 边界 |
| RFC-0007.2 | Production Architecture | 本 RFC 定义 Runtime 在 Production 中的位置 |
| RFC-0012 | Federation Layer | 本 RFC 定义 Runtime 如何消费 Federation 产出 |

---

## 11. Version

| 属性 | 值 |
|------|-----|
| RFC ID | RFC-0013 |
| Title | Runtime Boundary Enforcement |
| Version | 1.0.0 |
| Status | Frozen |
| Foundation Freeze | v3.4 |
| Protocol Compatibility | AISE 2.0.0-frozen |
| Scope | Protocol Layer — Runtime Enforcement Contract |