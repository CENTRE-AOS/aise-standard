# CENTRE Artifact Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-21
> 适用范围: CENTRE Foundation v3.2.0
> 前置: Runtime Contract v1.0.0, Skill Contract v1.0.0
> 关联: RFC-0011 Bootstrap Lifecycle, artifact-authority-map.json

---

## 1. 定义

CENTRE Artifact Contract 定义 CENTRE 生态系统中 Artifact 的权威模型。Artifact 是脱离 Source Repository 独立存在的不可变系统资产，是 Runtime Instance 的唯一合法安装源。

**Artifact 不是 Source。Source 是 Artifact 的输入，不是 Artifact。**

```
Source ≠ Artifact ≠ Runtime Instance
```

## 2. Artifact 五层 Authority Model

Artifact Civilization 按 Authority 分为五层，每层有独立的 Owner、职责和边界：

```
A0 Protocol Definition
        │
        │ defines
        ▼
A1 Source Authority
        │
        │ provides input
        ▼
A2 Build Authority
        │
        │ produces artifact
        ▼
A3 Artifact Authority
        │
        │ distributes
        ▼
A4 Install Authority
        │
        │ creates instance
        ▼
    Runtime Instance
```

### 2.1 A0 Protocol Definition

| 属性 | 值 |
|------|-----|
| Owner | aise-standard |
| Authority | PROTOCOL_DEFINITION |
| 职责 | 定义 Artifact Schema、Manifest Schema、Version Rules、Compatibility Rules |
| 产出 | artifact-contract.md, artifact schema, manifest schema |
| 禁止 | 生成 Runtime Artifact、执行 Build |

### 2.2 A1 Source Authority

| 属性 | 值 |
|------|-----|
| Owner | aos-runtime, aise-standard, aos-installer |
| Authority | SOURCE_AUTHORITY |
| 职责 | 提供 Source Input for Build |
| 产出 | Source Repository (git) |
| 禁止 | 直接发布 Artifact、绕过 Build 分发 |

### 2.3 A2 Build Authority

| 属性 | 值 |
|------|-----|
| Owner | aos-factory-new |
| Consumer of | A1 Source Authority |
| Authority | BUILD_AUTHORITY |
| 职责 | 编译、打包、生成 Checksum、生成 Artifact Manifest |
| 输入 | Source Repository |
| 产出 | Artifact Candidate (.pkg, manifest, checksum) |
| 禁止 | 修改 Protocol、修改 Runtime Contract、修改 Source |

### 2.4 A3 Artifact Authority

| 属性 | 值 |
|------|-----|
| Owner | Artifact Registry |
| Authority | ARTIFACT_AUTHORITY |
| 职责 | 管理 Artifact Manifest、Checksum、Signature、Version |
| 产出 | Signed Artifact |
| 禁止 | 重新生成 Source、修改 Artifact 内容 |

### 2.5 A4 Install Authority

| 属性 | 值 |
|------|-----|
| Owner | aos-installer |
| Authority | INSTALL_AUTHORITY |
| 职责 | 验证 Artifact 完整性、安装到 Runtime Instance、激活 |
| 输入 | Signed Artifact |
| 产出 | Runtime Instance |
| 禁止 | git clone source、npm install source、pip install source、直接修改 Runtime |

## 3. Artifact Manifest Schema

每个 Artifact 必须携带 `artifact.manifest.json`：

```json
{
  "artifact": {
    "artifact_id": "string",
    "artifact_version": "string",
    "artifact_type": "protocol | runtime | cli | skill-bundle",
    "artifact_state": "candidate",
    "build_time": "ISO8601",
    "origin": {
      "repository": "string",
      "source_type": "runtime | protocol | cli | skill-bundle",
      "commit": "string"
    },
    "authority": {
      "level": "A2",
      "issuer": "aos-factory-new"
    },
    "checksum": {
      "algorithm": "sha256",
      "hash": "string"
    },
    "signature": {
      "algorithm": "rsa-sha256",
      "issuer": "string",
      "value": "string"
    },
    "path": "string",
    "platform": "string"
  },
  "version_model": {
    "protocol_version": "string",
    "runtime_version": "string",
    "cli_version": "string",
    "artifact_version": "string",
    "independent": true,
    "note": "Protocol Version ≠ Runtime Version ≠ CLI Version ≠ Artifact Version"
  },
  "compatibility": {
    "protocol_min": "string",
    "protocol_max": "string",
    "runtime_min": "string",
    "runtime_max": "string"
  },
  "bootstrap_contract": {
    "required_files": ["AGENTS.md", "AGENT_CONTEXT.md"],
    "bootstrap_contract_version": "1.0"
  },
  "immutable": true
}
```

### 3.1 必填字段

| 字段 | 说明 |
|------|------|
| `artifact_id` | Artifact 唯一标识 |
| `artifact_version` | Artifact 版本号 |
| `artifact_type` | 类型：protocol / runtime / cli / skill-bundle |
| `artifact_state` | 生命周期状态：candidate / signed / verified / installed |
| `origin.commit` | 构建源 commit SHA |
| `origin.repository` | 源仓库名称 |
| `origin.source_type` | 源类型：runtime / protocol / cli / skill-bundle |
| `authority.level` | Artifact Authority 层级 |
| `authority.issuer` | 签发者身份 |
| `checksum.hash` | SHA256 哈希 |
| `version_model` | 版本模型声明（必须 `independent: true`） |
| `immutable` | 必须为 `true` |

### 3.2 版本模型约束

```
Protocol Version ≠ Runtime Version ≠ CLI Version ≠ Artifact Version
```

四个版本号完全独立，不可绑定、不可合并、不可互相推导。

| 版本 | 定义者 | 含义 |
|------|--------|------|
| protocol_version | aise-standard | 协议规范版本 |
| runtime_version | aos-runtime | Runtime 执行引擎版本 |
| cli_version | aos-installer | CLI 工具版本 |
| artifact_version | Build Authority | 本 Artifact 的发布版本 |

Artifact 仅**引用**这些版本，不修改它们。

### 3.3 artifact_state 生命周期状态

`artifact_state` 描述 Artifact 当前所处的生命周期阶段。状态转换由 Authority 边界控制。

| 状态 | 含义 | 设置者 | 下一状态 |
|------|------|--------|---------|
| `candidate` | Artifact Candidate，由 A2 Build Authority 产生，未签名 | A2 Build Authority | `signed` |
| `signed` | 已由 A3 Artifact Authority 签名，可分发 | A3 Artifact Authority | `verified` |
| `verified` | 已由 A4 Install Authority 验证完整性，可安装 | A4 Install Authority | `installed` |
| `installed` | 已安装到 Runtime Instance | A4 Install Authority | (终态) |

**状态转换规则**：

```
candidate → signed     (A3: Sign)
signed    → verified   (A4: Validate)
verified  → installed  (A4: Install)

禁止:
  candidate → verified (跳过签名)
  signed    → installed (跳过验证)
  candidate → installed (跳过签名和验证)
  installed → *        (不可逆)
```

**不可逆原则**：
- 状态只能向前，不能回退
- `installed` 是终态，不可转换为其他状态
- 任何状态变更必须通过对应 Authority 执行

### 3.5 Artifact Lifecycle vs Runtime Lifecycle（重要边界）

`artifact_state` 描述的是 **Artifact 的可用性状态**（Package Availability），不是 Runtime Instance 的运行状态。

```
Artifact Lifecycle (artifact_state):
  candidate → signed → verified → installed (终态)


Runtime Lifecycle (独立模型):
  created → initialized → active → suspended → terminated
```

**关键区别**：

| 维度 | Artifact Lifecycle | Runtime Lifecycle |
|------|-------------------|-------------------|
| 描述 | Package 在系统中的可用性 | Runtime Instance 的运行状态 |
| Owner | Artifact Authority Chain (A2/A3/A4) | Runtime Instance |
| `installed` 含义 | Package 已部署到 AOS_HOME | N/A（不属于 Runtime 状态） |
| 终态 | `installed`（不可逆） | `terminated`（可重建） |
| 定义位置 | artifact-contract.md | artifact-consumption-contract.md |

**禁止混合**：

```
FORBIDDEN:
  Runtime State = Artifact State
  artifact_state = "active"      (active 是 Runtime 状态，不是 Artifact 状态)
  runtime_state = "installed"    (installed 是 Artifact 状态，不是 Runtime 状态)
```

`installed` 是 Artifact 的终点，也是 Runtime 的起点。两者在同一时间点交汇，但描述的是不同对象的不同维度。

### 3.4 State Mutation Contract

`artifact_state` 的变更权属于当前状态对应的 Authority。**禁止直接修改 `artifact_state` 字段**——状态变更必须是 Authority Action 的结果。

| State | Creator | 当前持有者 | 可变更者 | 变更动作 | 目标状态 |
|-------|---------|-----------|---------|---------|---------|
| `candidate` | A2 Build Authority | A2 | A3 only | Sign | `signed` |
| `signed` | A3 Artifact Authority | A3 | A4 only | Verify | `verified` |
| `verified` | A4 Install Authority | A4 | A4 only | Install | `installed` |
| `installed` | A4 Install Authority | Runtime Instance | 无 | (终态) | (不可变) |

**State Mutation 禁止规则**：

```
FORBIDDEN:

  # 直接修改 state 字段
  modify artifact_state without Authority Action

  # 跨 Authority 修改
  A2 change state from signed → candidate  (A2 不拥有 signed)
  A4 change state from candidate → signed   (A4 不拥有 candidate)
  A3 change state from verified → signed    (A3 不拥有 verified)

  # 跳过 Authority Action
  candidate → verified  (跳过 Sign)
  signed → installed    (跳过 Verify)
  candidate → installed (跳过 Sign + Verify)

  # 回退
  signed → candidate
  verified → signed
  installed → *
```

**核心原则**：

```
State Transition MUST be caused by Authority Action.
State Mutation IS FORBIDDEN.

State 是 Authority Action 的产物，不是可独立修改的字段。
```

## 4. Artifact 生命周期

```
Source Repository
        │
        │ Build (A2)
        ▼
Artifact Candidate     [state: candidate]
        │
        │ Sign (A3)
        ▼
Signed Artifact        [state: signed]
        │
        │ Validate (A4)
        ▼
Verified Artifact      [state: verified]
        │
        │ Install (A4)
        ▼
Runtime Instance       [state: installed]
        │
        │ Activate (A4)
        ▼
Active Instance
```

### 4.1 不可逆原则

- Artifact 一旦签名发布，永不修改
- 任何修改必须通过新版本号发布
- 签名验证失败 → 终止安装

### 4.2 Install Boundary 硬约束

Install 只能通过 Artifact 安装：

```
ALLOWED:
  artifact → validate → install → runtime instance

FORBIDDEN:
  git clone source → runtime instance
  npm install source → runtime instance
  pip install source → runtime instance
  curl source → runtime instance
  runtime rebuild artifact
  runtime patch artifact
  runtime mutate installed artifact
```

任何绕过 Artifact 的安装都是对 Artifact Boundary 的破坏。Runtime 自更新（下载 source → patch 自身）是对 Artifact Civilization 的破坏。

## 5. 与现有 Contract 的关系

### 5.1 与 Runtime Contract 的关系

- Runtime Contract §2.1 CLI Interface 定义 `install` 命令 — 该命令必须通过 Artifact 安装，不可直接安装 Source
- Runtime Contract §2.4 Registry Interface — 需新增 Artifact Registry
- Runtime Contract §3 版本模型 — Artifact Contract 扩展为四线独立

### 5.2 与 Skill Contract 的关系

- Skill Contract §3.1 Skill Manifest 定义 Skill 的 `produced_by` — Artifact 的 `produced_by` 是 Build Authority
- Skill Contract §4 #2 "Skill 不定义协议" — Artifact Contract 同理，Artifact 不定义协议
- Skill 安装（`centre skills install`）必须通过 Artifact，不可绕过

### 5.3 与 Bootstrap Lifecycle 的关系

- Bootstrap 在 Runtime Instance 启动时执行，Bootstrap 需要知道当前 Runtime Instance 来自哪个 Artifact
- Artifact Manifest 的 `bootstrap_contract` 字段声明了 Bootstrap 所需的文件

## 6. 不可违背约束

1. Source ≠ Artifact — Artifact 脱离 Source 独立存在
2. Artifact 不可变 — 一旦签名发布，永不修改
3. Install 不可绕过 Artifact — 禁止 git clone / npm install / pip install 作为安装路径
4. 版本模型独立 — Protocol Version ≠ Runtime Version ≠ CLI Version ≠ Artifact Version
5. Runtime 不生产 Artifact — Runtime 只消费 Artifact
6. Protocol 定义 Artifact Contract — Artifact 的法律层属于 aise-standard
7. Build 不修改 Source — Build Authority 只能读取 Source，不能修改
8. Artifact 必须携带 Manifest — 无 Manifest 的 Artifact 不可安装
9. Artifact 必须签名 — 无签名的 Artifact 不可安装
10. Artifact 不包含 Secrets — 密钥、凭证、私钥不属于 Artifact