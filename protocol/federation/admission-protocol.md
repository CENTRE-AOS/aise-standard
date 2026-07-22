# Federation Admission Protocol

> Protocol: CENTRE-FEDERATION v1.0.0
> Layer: Federation Governance
> Status: ACTIVE
> Created: 2026-07-21
> Incident Reference: Post-Federation Bootstrap — Repository Identity Resolution Failure
> Supersedes: N/A (new protocol)
> Related: Gate Contract 0-7, Bootstrap Contract v2.0.0-frozen, Identity Protocol v1.0

---

## 1. Purpose

定义 Agent 进入 Federation 多仓库体系时必须执行的 **Repository Identity Resolution** 流程。

回答 Agent 在 Federation 层面临的第一个问题：

> **Given a directory, which Authority does it belong to?**

Federation Admission Protocol 确保 Agent 在跨仓库操作中，不会将正确动作作用于错误对象。

---

## 2. Architecture Position

Federation Layer 是 CENTRE 架构中此前缺失的独立层：

```
                     CENTRE Architecture

                          │

                  Protocol Layer
                 (AISE Standard)
                          │

                  Federation Layer       ← 本 Protocol 定义
            (Repository Identity + Admission)
                          │

                  Runtime Layer
            (Gateway Runtime Execution)
                          │

                  Project Layer
            (Factory / Runtime / Installer)
                          │

                  Agent Layer
```

**为什么 Federation Layer 必须独立？**

- Protocol 解决：什么规则？
- Runtime 解决：怎么执行？
- **Federation 解决：这个仓库是谁？**

没有 Federation Layer 时，Agent 看到的是 `filesystem name`，而不是 `authority identity`。这导致了 2026-07-21 事故：`aos-factory`（transition monorepo）被错误识别为 A2 Build Authority。

---

## 3. Core Principle

```
Identity Manifest > Folder Name > Git Remote Name
```

优先级链：

| Priority | Source | Authority Weight |
|:--------:|--------|:---------------:|
| 1 | Protocol Identity（`.project/centre.protocol.json`） | HIGHEST |
| 2 | Project Manifest（`project.declaration.json` / `factory.manifest.json`） | HIGH |
| 3 | Remote Identity（`.github/remote-identity.json`） | MEDIUM |
| 4 | Filesystem Name（directory name） | LOWEST — 仅供参考，不用于身份判断 |

Agent 在 Federation 层必须始终按此优先级链解析身份，不得跳过。

---

## 4. Federation Admission Gates

Federation Admission 在现有 Gate 0 (Identity Check) 之前增加一个跨仓库层：

```
Federation Layer (CENTRE-FEDERATION v1.0.0):
     FG-0: Repository Discovery Gate
     FG-1: Identity Verification Gate
     FG-2: Authority Admission Gate
              │
              │ (passed)
              ▼
     FG-3: Bootstrap Instantiation Gate
              │
              │ (passed)
              ▼
     FG-4: Federation Certificate Gate

Existing Agent Layer (gate-contract.md):
     Gate 0: Identity Check (Agent-level)
     Gate 1: Project Admission (Project-level)
     ...
```

---

## 5. FG-0: Repository Discovery Gate

**When**: Agent 首次进入 Federation 目录树，或跨仓库操作前。

**What**:
1. 发现当前目录是否为 Git 仓库（检查 `.git/` 存在性）
2. 识别仓库类型：

| 类型 | 判定条件 | 操作权限 |
|------|----------|:------:|
| Federation Repository | 包含 `.github/remote-identity.json` | Full (after FG-1/FG-2) |
| Standard Project | 包含 `.agent-entry.json` 或 `.project/centre.protocol.json` | Standard Gate 0-7 |
| Non-CENTRE Repository | 无上述文件 | Read-Only |
| Archive Artifact | 位于 `archive/` 或包含 `ARCHIVE_MANIFEST.json` | Read-Only |

**Output**: Repository Type Classification

**Failure**: 若无法确定仓库类型 → 标记为 UNKNOWN，Agent 不得执行任何 Mutation 操作，仅允许 Read。

**禁止**:
- 根据目录名判断类型
- 根据 Git Remote URL 判断类型（有可能多个 repo 指向同一 remote）

---

## 6. FG-1: Identity Verification Gate

**核心 Gate**。回答：**"Who are you?"**

**When**: FG-0 识别为 Federation Repository 后。

**What** — 按优先级执行身份解析链：

### Step 1: Read Protocol Identity
读取 `.project/centre.protocol.json`，提取：
- `protocol_id` — 必须为 `"AISE"`
- `protocol_version` — 必须兼容 Agent 的 Protocol 版本
- `project_id` — 项目唯一标识
- `project_role` — 项目角色声明

### Step 2: Read Project Manifest
读取 `project.declaration.json` 或 Authority-specific manifest（如 `factory.manifest.json`），验证：
- `type` — 项目类型（不得为 `transition_monorepo` 或 `archive`）
- `is_production_source` — 必须为 `true` 才能接受 Bootstrap
- `frozen` — 若为 `true`，项目已冻结，拒绝 Bootstrap

### Step 3: Read Remote Identity
读取 `.github/remote-identity.json`，验证：
- `authority` — Authority 类型
- `authority_level` — Authority 层级（A0/A1/A2/A3/A4）
- `lifecycle` — 生命周期状态（`production` / `archived` / `transition`）
- `remote_status` — 远程状态（`active` / `inactive`）

### Step 4: Cross-Validate
- `remote-identity.json` 中的 `authority` 必须与 project manifest 中的 `type` 一致
- `project_id` 必须在所有文件中一致
- `local_path` 不是身份来源，仅作为调试信息

**Output**: Verified Repository Identity

**Failure Conditions**:
- `remote-identity.json` 不存在或无效 → HALT，报告 "FG-1: No Identity Manifest"
- `lifecycle` 为 `archived` 或 `transition` → 标记 READ-ONLY，HALT
- `is_production_source` 为 `false` → HALT，报告 "FG-1: Not Production Source"
- 交叉验证失败 → HALT，报告 "FG-1: Identity Cross-Validation Failed"

---

## 7. FG-2: Authority Admission Gate

**When**: FG-1 通过后。

确认：**"Does this Repository belong in this Federation?"**

**What**:
1. 确认 `authority_level` 与当前 Agent 的 Mission 目标匹配：
   - A0 Agent 只能 Bootstrap A0 (Protocol) 仓库
   - A1 Agent 只能 Bootstrap A1 (Runtime) 仓库
   - A2 Agent 只能 Bootstrap A2 (Factory) 仓库
   - A4 Agent 只能 Bootstrap A4 (Distribution) 仓库
2. 验证 Protocol 兼容性：
   - `.project/centre.protocol.json` 中的 `protocol_version` 必须与 Agent 声明的 Protocol 版本兼容
   - `.project/centre.protocol.json` 中的 `runtime_version` 必须在 Agent Runtime 兼容范围内
3. 验证 Dependency Lock：
   - `.github/dependency-lock.json` 的 `upstream` 声明与实际 Federation 链路一致
   - `forbidden_consumers` 不包含当前 Agent
4. 验证 Authority Boundary：
   - 仓库声明的 `owns` / `does_not_own` 必须与 Federation 中该 Authority 的标准定义一致
   - 不得出现跨 Authority 的 owns 声明（如 Factory 声称 owns `runtime`）

**Output**: Authority Admission Granted or Denied

**Failure Conditions**:
- Authority mismatch → HALT，报告 "FG-2: Authority Boundary Violation"
- Protocol 版本不兼容 → HALT，报告 "FG-2: Protocol Version Incompatible"
- Dependency Lock 违规 → HALT，报告 "FG-2: Dependency Lock Violation"
- Authority Boundary 异常 → HALT，报告 "FG-2: Authority Boundary Cross-Declaration"

---

## 8. FG-3: Bootstrap Instantiation Gate

**When**: FG-2 通过后。Agent 已确认仓库身份，准备执行初始化或同步。

**What** — 对缺失文件进行 Protocol Template 实例化：

1. 检查 Bootstrap Contract 必检文件集（参见 `bootstrap-contract.md`）
2. 对缺失文件，从 `aise-standard/templates/` 实例化：
   - `.agent-entry.json` ← `.agent-entry.json.example`
   - `.project/centre.protocol.json` ← Protocol manifest schema
   - `AGENTS.md` ← Authority-specific template
   - `AGENT_BOOTSTRAP.md` ← Authority-specific bootstrap template
   - `PROJECT_CONTEXT.md` ← Authority-specific context template
3. 验证所有实例化文件的字段完整性
4. 执行 Version Alignment（确认所有 manifest 中的 Protocol/Runtime/Foundation 版本一致）

**Output**: Instantiated Bootstrap Files

**Failure Conditions**:
- Template 不存在 → HALT，报告 "FG-3: Template Missing"
- 实例化后字段不完整 → HALT，报告 "FG-3: Instance Incomplete"
- Version Alignment 失败 → HALT，报告 "FG-3: Version Inconsistent"

**禁止**:
- 项目自行创建 Protocol Layer 文件（必须从 aise-standard 模板实例化）
- 创建 Protocol 未定义的字段
- 修改不属于当前 Authority 的文件

---

## 9. FG-4: Federation Certificate Gate

**When**: FG-3 通过后。

定义：`PROJECT_BOOTSTRAP_READY.md` **不是**"项目初始化完成标记"，而是 **Project Admission Certificate**——项目获得 Federation 身份的正式凭证。

**What**:
1. 为当前 Repository 生成 `PROJECT_BOOTSTRAP_READY.md`
2. 内容必须包含：

| Section | 说明 |
|---------|------|
| Identity | Repository name (+ remote), Authority Level, Version, Foundation Freeze |
| Authority | Owns / Does NOT Own / Is NOT — 明确的边界声明 |
| Federation Admission Gates | FG-0/FG-1/FG-2 通过状态 |
| Owned Resources | 具体目录/文件路径列表 |
| Forbidden Resources | 具体不可修改的路径/域 |
| Dependencies | Upstream (Provider + Repo + Version) + Downstream (Consumer + Repo + Version) |
| Next Execution Phase | Agent 下一步执行的动作 |

3. Certificate 格式见 `aise-standard/templates/project-bootstrap/PROJECT_BOOTSTRAP_READY.template.md`
4. Certificate 详细语义定义见 `federation-certificate.md`

**Output**: `PROJECT_BOOTSTRAP_READY.md` — Project Admission Certificate

**Failure Conditions**:
- Certificate 字段缺失 → HALT
- Authority 声明与 FG-1 验证结果不一致 → HALT

---

## 10. Complete Gate Flow

```
FG-0: Repository Discovery
     │
     ├── Archive Artifact? → READ-ONLY, STOP
     ├── Non-CENTRE?       → READ-ONLY, STOP
     ├── UNKNOWN?          → READ-ONLY, STOP
     │
     ▼ (Federation Repository)
FG-1: Identity Verification
     │
     ├── No Manifest?          → HALT
     ├── Transition/Archived?  → HALT
     ├── Not Production?       → HALT
     │
     ▼
FG-2: Authority Admission
     │
     ├── Authority Mismatch?       → HALT
     ├── Protocol Incompatible?    → HALT
     ├── Dependency Lock Violation?→ HALT
     │
     ▼
FG-3: Bootstrap Instantiation
     │
     ├── Template Missing?     → HALT
     ├── Instance Incomplete?  → HALT
     ├── Version Inconsistent? → HALT
     │
     ▼
FG-4: Federation Certificate
     │
     │ (PROJECT_BOOTSTRAP_READY.md generated)
     │
     ▼
Gate 0: Identity Check (Agent-level)
     ↓
Gate 1: Project Admission (Project-level)
     ↓
... (continues per gate-contract.md)
```

任一步失败，终止后续流程。

---

## 11. Archive Artifact Handling

标记为 `archive/` 或包含 `ARCHIVE_MANIFEST.json` 的目录为 Archive Artifact，不参与 Federation。

Agent 发现 Archive Artifact 时：
- 仅允许 Read 操作
- 禁止任何 Mutation（写入、修改、删除）
- 禁止将其作为 Authority Repository 进行 Bootstrap
- 如需要其中历史代码，必须通过 Archive Reference（引用，不操作）

---

## 12. Incident Record: 2026-07-21 Repository Identity Resolution Failure

### Incident

Post-Federation Bootstrap 阶段，Agent 将 `aos-factory`（transition monorepo，`is_production_source: false`）错误识别为 A2 Build Authority，执行了 Bootstrap 文件写入。

### Root Cause

缺少 Federation Layer。Agent 以目录名推测身份，未读取 `project.declaration.json` 验证 `is_production_source`。

### Architectural Impact

此事故验证了 CENTRE 的核心设计：**如果没有 Identity Layer，这只是一次路径误判。有 Identity Layer 后，它被识别为 Authority Boundary Violation。** 同时暴露了 Federation Layer 的缺失——Protocol → Runtime 模型缺少跨仓库身份解析层。

### Resolution

1. FG-0/FG-1/FG-2/FG-3/FG-4 正式规则化为此 Protocol
2. `aos-factory` → `archive/aos-factory-transition/`（归档）
3. `aos-factory-new` 确认为真正的 A2 Factory（remote-identity.json 确认）
4. 错误写入被撤销
5. Federation Admission Certificate 定义正式化

### Prevention

此 Protocol 正式化后，所有 CENTRE Agent 在 Federation 层操作前必须执行完整 FG-0 → FG-4 验证链。对于 Factory Agent 本身，也必须在 Build 前执行 FG-0/FG-1/FG-2。

---

## 13. Impact on Factory

Factory Agent 的 Build 流程必须包含 Federation Gate：

**错误（旧）**:
```
scan folder → build
```

**正确（新）**:
```
Repository Discovery (FG-0)
     ↓
Identity Verification (FG-1)
     ↓
Authority Admission (FG-2)
     ↓
Artifact Build
```

Factory 本身也需要 Federation Gate，因为它同样需要确认：输入源仓库的身份是正确的。

---

## 14. remote-identity.json v2.0 Recommendation

当前 `remote-identity.json` 定位不够强。建议在后续 RFC 中升级：

### Current (v1.x)

```json
{
  "local_path": "aos-factory-new",
  "remote": "CENTRE-AOS/aos-factory"
}
```

### Proposed (v2.0)

```json
{
  "federation_identity": {
    "id": "centre.a2.factory",
    "authority": "A2",
    "role": "Build Authority"
  },
  "repository": {
    "remote": "CENTRE-AOS/aos-factory",
    "local_alias": "aos-factory-new"
  },
  "status": "active"
}
```

关键变更：`federation_identity` 作为独立字段，Agent 无需推断身份。

---

## 15. Relationship to Existing Protocols

| Protocol | Scope | Layer |
|----------|-------|-------|
| Identity Protocol | 单项目身份声明（`.agent-entry.json`） | Project |
| Gate Contract (0-7) | 单项目 Agent 准入 | Project |
| **Federation Admission Protocol** | **跨仓库 Repository Identity Resolution + Bootstrap + Certificate** | **Federation** |
| Federation Identity Resolution | 身份解析优先级规则 | Federation |
| Federation Certificate | PROJECT_BOOTSTRAP_READY 正式定义 | Federation |
| Bootstrap Contract | 生产仓库最小文件集定义 | Federation |

Federation Admission Protocol **在** Identity Protocol + Gate Contract **之前**执行。

---

## 16. Version

| 属性 | 值 |
|------|-----|
| Protocol Name | CENTRE-FEDERATION Admission Protocol |
| Version | 1.0.0 |
| Status | ACTIVE |
| Gates | FG-0, FG-1, FG-2, FG-3, FG-4 |
| Foundation Compatibility | CENTRE v3.2.0 |
| Protocol Compatibility | AISE 2.0.0-frozen |
