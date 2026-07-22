# RFC-0012: Federation Layer Introduction

> RFC ID: RFC-0012
> Title: Federation Layer — Identity Resolution & Admission between Protocol and Runtime
> Version: 1.0.0
> Status: Frozen
> Type: Architecture Decision Record (ADR)
> Date: 2026-07-21
> Author: CENTRE Protocol Authority
> Category: Constitutional
> Requires: RFC-0002 (Kernel Contract), RFC-0007.2 (Production Architecture)
> Scope: aise-standard (Protocol Layer)
> Freeze Target: CENTRE Foundation Freeze v3.3

---

## 1. Abstract

引入 CENTRE 架构的第五层——**Federation Layer**，位于 Protocol Layer 和 Runtime Layer 之间。

Federation Layer 解决跨仓库 Agent 操作的第一个问题：

> **Given a directory, which Authority does it belong to?**

它是 CENTRE 生态的"组织身份系统"——类似 DNS 的 name resolution、TLS certificate chain、Kubernetes admission controller，但服务于 Agent Ecosystem。

---

## 2. Motivation

### 2.1 问题：Identity Boundary Problem

CENTRE 架构原有模型：

```
Protocol → Runtime → Project
```

Agent 在跨仓库操作时，依赖 **filesystem name** 判断身份：

```
错误的身份解析：
  Scan folder "aos-factory"
       ↓
  Assume: "This is A2 Factory"
       ↓
  Bootstrap with A2 Factory rules
```

这导致 2026-07-21 事故：`aos-factory`（`transition_monorepo`, `is_production_source: false`）被错误识别为 A2 Build Authority，执行了 Bootstrap 文件写入。

### 2.2 根因：架构缺失层

Protocol 定义了"什么规则"，Runtime 定义了"怎么执行"。但缺少一个中间层回答：

> **"这个仓库是谁？"**

目录名不能作为身份。Git remote 不能作为身份。路径不能作为身份。必须有一个独立的 Identity Resolution 层。

### 2.3 影响范围

如果不在架构层面解决，未来 Federation 扩展到 100+ 仓库、1000+ Agent 时，类似问题会变成：

> Agent 自动进入错误 Authority，修改错误仓库。

这类错误比代码 bug 更危险——它是 **Authority Boundary Violation**。

---

## 3. Decision

### 3.1 引入 Federation Layer

CENTRE 架构从四层扩展为五层：

```
                    CENTRE

              Protocol Constitution
                      │
                      │ "什么规则？"
                      ▼
              Federation Layer          ← NEW
          Identity / Admission / Certificate
                      │
                      │ "这个仓库是谁？"
                      ▼
              Gateway Runtime
       Execute / Route / Lifecycle / Enforcement
                      │
                      │ "怎么执行？"
                      ▼
              Project Runtime
     Context / Memory / State / Artifact
                      │
                      ▼
                 Agent Runtime
          Skill / Task / Intelligence Loop
```

### 3.2 Federation Layer 定位

**Federation 是 Protocol Layer 的子领域，不是 Runtime 模块。**

```
CENTRE Protocol
│
├── Constitution
├── Identity
├── Federation          ← 本 RFC 定义的子领域
├── Context
├── Asset
└── Governance
        │
        │ consumed by
        ▼
CENTRE Runtime
```

关键约束：

| 正确 | 错误 |
|------|------|
| Federation = Protocol | Federation Runtime |
| Gateway = Executor | Federation Manager / Server / Database |
| Project = Owner | CENTRE owns ecosystem |

### 3.3 核心原则

```
Identity Manifest > Folder Name > Git Remote Name
```

优先级链：

| Priority | Source | Weight |
|:--------:|--------|:------:|
| 1 | Protocol Identity (`.project/centre.protocol.json`) | HIGHEST |
| 2 | Project Manifest (`project.declaration.json`) | HIGH |
| 3 | Remote Identity (`.github/remote-identity.json`) | MEDIUM |
| 4 | Filesystem Name | LOWEST — 仅供参考 |

### 3.4 Federation ≠ 中心化平台

Federation 最容易走偏的方向：

```
❌ Federation Manager
❌ Federation Server
❌ Federation Database
❌ CENTRE owns ecosystem
```

正确模型：

```
✅ Federation = Protocol
✅ Gateway = Executor
✅ Project = Owner
✅ CENTRE defines: who can enter, how to identify, how to establish trust
```

---

## 4. Federation Admission Gates

Federation Layer 定义 5 个 Admission Gate，在 Runtime Gate 0-7 之前执行：

```
FG-0: Repository Discovery
     │  分类仓库类型 (Federation/Standard/Archive/Unknown)
     ▼
FG-1: Identity Verification
     │  CORE: 身份解析优先链验证
     ▼
FG-2: Authority Admission
     │  Authority 匹配 + Protocol 兼容 + Dependency Lock
     ▼
FG-3: Bootstrap Instantiation
     │  Template → Instance (仅 aise-standard 模板)
     ▼
FG-4: Federation Certificate
     │  PROJECT_BOOTSTRAP_READY.md = Project Admission Certificate
     ▼
Gate 0-7: Agent Admission (gate-contract.md)
```

### 4.1 FG-0: Repository Discovery Gate

| 类型 | 判定 | 操作权限 |
|------|------|:------:|
| Federation Repository | 含 `.github/remote-identity.json` | Full (after FG-1/2) |
| Standard Project | 含 `.agent-entry.json` | Standard Gate 0-7 |
| Non-CENTRE | 无上述文件 | Read-Only |
| Archive Artifact | 在 `archive/` 或含 `ARCHIVE_MANIFEST.json` | Read-Only |
| UNKNOWN | 无法确定 | Read-Only, HALT |

### 4.2 FG-1: Identity Verification Gate

**核心 Gate。** 按优先级链执行身份解析：

1. Read `.project/centre.protocol.json` — `protocol_id`, `project_id`, `project_role`
2. Read project manifest — `type`, `is_production_source`
3. Read `.github/remote-identity.json` — `authority`, `authority_level`, `lifecycle`
4. Cross-validate all three sources

**Failure conditions**:
- `is_production_source: false` → HALT
- `lifecycle: archived/transition` → HALT
- Cross-validation mismatch → HALT

### 4.3 FG-2: Authority Admission Gate

1. A0 Agent → only A0 repos; A1 → only A1; A2 → only A2; A4 → only A4
2. Protocol version compatibility check
3. Dependency lock verification
4. Authority boundary validation (no cross-authority owns declarations)

### 4.4 FG-3: Bootstrap Instantiation Gate

对缺失文件从 `aise-standard/templates/` 实例化。禁止项目自行创建 Protocol Layer 文件。

### 4.5 FG-4: Federation Certificate Gate

`PROJECT_BOOTSTRAP_READY.md` 正式定义为 **Project Admission Certificate**——三方契约 (Protocol Authority ↔ Certificate ↔ Agent)。

---

## 5. Impact Analysis

### 5.1 Protocol Layer

- 新增 `protocol/federation/` 子领域
- 新增 3 个 Protocol 文档：admission-protocol.md, identity-resolution.md, federation-certificate.md
- 不影响现有 Protocol Contracts

### 5.2 Runtime Layer

- **无 breaking change**
- Runtime 只接受 Federation Verified Entity，不判断身份
- 未来增加 Federation Verification Hook（单一入口，不改变 Runtime 核心）

### 5.3 Factory Layer

Factory Agent 的 Build 流程必须包含 Federation Gate：

```
Repository Discovery (FG-0) → Identity Verification (FG-1) → Authority Admission (FG-2) → Artifact Build
```

### 5.4 Installer Layer

Installer 分发前必须验证目标仓库的 Federation Admission Certificate。

### 5.5 Foundation Freeze

本 RFC 触发 CENTRE Foundation Freeze v3.3，冻结：
1. Federation Contract
2. Identity Resolution Algorithm
3. Certificate Format
4. Versioning Rule

### 5.6 生态影响

Federation Layer 打开了生态入口：

```
以前：AOS Runtime 只能管理自己的项目

现在：CENTRE Federation
      ├── Project A
      ├── Project B
      ├── External Agent
      ├── External Tool
      └── External Runtime
      统一 Admission
```

类比 Linux Kernel + ABI + Distribution，Git Repository + Protocol + Remote，Kubernetes API Contract + Admission + Controller。

---

## 6. remote-identity.json v2.0

当前 `remote-identity.json` 定位不够强。建议在后续 RFC 中升级为 v2.0：

```json
{
  "identity": {
    "federation_id": "centre.a2.factory",
    "entity_type": "repository",
    "owner": "CENTRE-AOS",
    "origin": "aise-standard",
    "trust_level": "production"
  },
  "binding": {
    "protocol_version": "2.0.0-frozen",
    "runtime_requirement": ">=3.2.0"
  },
  "certificate": {
    "status": "active",
    "issued_at": "2026-07-21",
    "issuer": "CENTRE-FEDERATION v1.0.0"
  }
}
```

**关键约束：Identity ≠ State。** 禁止加入 runtime state、memory、capability list 等运行时状态字段。

---

## 7. Incident Reference

**2026-07-21 Repository Identity Resolution Failure**

Agent 将 `aos-factory`（`transition_monorepo`, `is_production_source: false`）错误识别为 A2 Build Authority。

此事故暴露了 Federation Layer 的缺失，直接催生了 CENTRE-FEDERATION v1.0.0 和本 RFC。

---

## 8. Alternatives Considered

### 8.1 在 Runtime 中实现 Identity Resolution

**Rejected**: 会导致 Runtime 耦合仓库身份判断逻辑。Runtime 应只执行，不判断身份。

### 8.2 在 Agent 层实现 Identity Resolution

**Rejected**: 每个 Agent 独立实现会导致不一致。Identity Resolution 必须是 Protocol 标准。

### 8.3 不引入新层，增强现有 Gate 0

**Rejected**: Gate 0 是单项目 Agent 准入，不处理跨仓库 Federation 场景。两个问题域不同，不应合并。

---

## 9. Version

| 属性 | 值 |
|------|-----|
| RFC ID | RFC-0012 |
| Title | Federation Layer Introduction |
| Version | 1.0.0 |
| Status | Frozen |
| Foundation Freeze | v3.3 |
| Protocol Compatibility | AISE 2.0.0-frozen |
| CENTRE-FEDERATION | v1.0.0 |