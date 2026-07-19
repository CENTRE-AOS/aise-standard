# AOS System Skill Architecture Specification

> 版本: 3.0.0frozen
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: AOS System Skill Layer v3.0
> 前置: CENTRE Gateway Runtime Constitution v1.0.0frozen

---

## Chapter 0 — Prime Directive

### 0.1 优先级法则

AOS System Skill 的优先级继承 CENTRE Runtime Constitution：

```
CENTRE Architecture Principles
    >
AISE Protocol
    >
System Skill Contract
    >
Skill Implementation
    >
Agent Invocation
```

Skill 是 Runtime 的扩展，不是 Agent 的工具。Skill 属于 Runtime，不属于 Agent。

### 0.2 不可违背原则

1. **Skill 归属 Runtime**：System Skill 是 AOS Runtime 的系统能力，不属于任何 Agent、Project、或协议定义
2. **Skill 不定义协议**：Skill 只执行，不定义。协议定义属于 aos-protocol-factory
3. **Skill 不拥有项目资产**：Skill 操作 Project 资产，但不拥有。所有权属于 Project
4. **Skill 不绕过 Runtime 接口**：Skill 必须通过 Frozen Interfaces（IIdentity/IContext/IEvent/ILifecycle/ICapability/IAdapter）获取上下文，不得自行收集
5. **Skill 不对 Agent 暴露内部实现**：Agent 只通过 State Machine 触发 Skill，不得直接调用 Skill 内部函数
6. **Skill 不可变**：安装后 manifest 不可修改，integrity hash 验证完整性
7. **Skill 不直接调用外部工具**：Git、SVN 等外部工具通过 ICapability 接口调用，Skill 不直接执行外部命令

### 0.3 System Skill 是什么

System Skill 不是普通脚本。它是 AOS Runtime 的**系统服务层（System Service Layer）**：

```
CENTRE Architecture
    │
    ├── AISE Protocol           ← 协议定义
    │
    ├── Gateway Runtime         ← 协议执行
    │   │
    │   ├── Frozen Interfaces   ← 系统契约（IIdentity/IAdmission/IContext/ICapability/IAdapter/IEvent）
    │   │
    │   ├── System Skills       ← 系统服务层（本章程定义）
    │   │
    │   ├── Skill Runtime       ← loader / validator / executor
    │   │
    │   ├── Command Router      ← Human Override → State Machine → Skill
    │   │
    │   └── CLI / Adapters      ← 表现层
    │
    └── Projects                ← 项目实例
```

### 0.4 System Skill 不是什么

| 不是 | 原因 |
|------|------|
| Agent 的私有工具 | Skill 属于 Runtime，不属于 Agent |
| 协议定义 | 协议由 aos-protocol-factory 定义 |
| 项目脚本 | Skill 不进入 Project 仓库 |
| 用户自定义扩展 | System Skill 是预装的、不可变的系统能力 |
| Adapter 实现 | Adapter 是连接层，Skill 是服务层 |

---

## Chapter 1 — Skill Identity

### 1.1 命名规范

```
格式: aise-<name>
示例: aise-admission, aise-handoff, aise-gitops
```

### 1.2 分类体系（编号前缀分层）

System Skills 按职责分为 6 层，使用编号前缀确保加载顺序：

| 编号 | 层 | 职责 | 技能 |
|------|------|------|------|
| 00-kernel | Kernel Security | 准入、身份、信任 | aise-admission |
| 10-lifecycle | Project Lifecycle | 项目生命周期 | aise-bootstrap, aise-handoff, aise-archive |
| 20-governance | Project Governance | 项目治理 | aise-gitops |
| 30-protocol | Protocol Sync | 协议同步 | aise-fetch, aise-sync |
| 40-adapter | Agent Adapter | Agent 适配注入 | aise-inject |
| 50-maintenance | System Maintenance | 系统维护 | aise-health, aise-update |

### 1.3 目录结构

```
aos-runtime/
└── system-skills/
    ├── manifest.schema.json          ← Skill Manifest Schema v3.0
    ├── registry.json                 ← Skill Registry（Runtime 加载入口）
    ├── 00-kernel/
    │   └── aise-admission/
    ├── 10-lifecycle/
    │   ├── aise-bootstrap/
    │   ├── aise-handoff/
    │   └── aise-archive/
    ├── 20-governance/
    │   └── aise-gitops/
    ├── 30-protocol/
    │   ├── aise-fetch/
    │   └── aise-sync/
    ├── 40-adapter/
    │   └── aise-inject/
    └── 50-maintenance/
        ├── aise-health/
        └── aise-update/
```

### 1.4 加载顺序

```
00-kernel → 10-lifecycle → 20-governance → 30-protocol → 50-maintenance → 40-adapter
```

Adapter 层最后加载，因为它依赖其他层的 Skill 已就绪。

---

## Chapter 2 — Skill Manifest

### 2.1 Manifest Schema v3.0（冻结字段）

每个 Skill 必须包含 `manifest.json`，定义以下字段：

```json
{
  "skill_id": "aise-<name>",
  "version": "1.0.0",
  "runtime_min": "2.2.0",
  "protocol": "AISE",
  "protocol_version": "2.6.0",
  "schema_version": "3.0",
  "state_triggers": ["agent.enter", "project.attach"],
  "immutable": true,
  "produced_by": {
    "specification": "aos-protocol-factory",
    "runtime": "aos-runtime"
  },
  "installed_at": "",
  "compatibility": {
    "runtime": ">=2.2 <3.0",
    "protocol": "AISE-1.x",
    "skill_api": "1.0"
  },
  "integrity": {
    "algorithm": "sha256",
    "hash": ""
  },
  "trust": {
    "issuer": "aos-runtime",
    "algorithm": "none"
  },
  "requires": [
    "IContext>=1.0",
    "IEvent>=1.0"
  ],
  "entry_point": "skill.ps1",
  "actions": ["validate", "issue"],
  "description": "..."
}
```

### 2.2 字段语义

| 字段 | 类型 | 必填 | 含义 |
|------|------|------|------|
| `skill_id` | string | 是 | 技能唯一标识 |
| `version` | string | 是 | 语义化版本 |
| `runtime_min` | string | 是 | 最低 Runtime 版本 |
| `protocol` | string | 是 | 关联协议 |
| `protocol_version` | string | 是 | 协议版本 |
| `schema_version` | string | 是 | Manifest Schema 版本，当前为 3.0 |
| `state_triggers` | array | 是 | 触发状态列表（不得包含自我触发的状态） |
| `immutable` | bool | 是 | 安装后不可修改 |
| `produced_by` | object | 是 | 技能来源（specification + runtime） |
| `installed_at` | string | 是 | 安装时间戳 |
| `compatibility` | object | 是 | 双向兼容性矩阵 |
| `integrity` | object | 是 | 完整性校验（SHA256 hash，非签名） |
| `trust` | object | 是 | 信任来源声明（v2.x 仅声明 issuer，v3.0+ 为 RSA/ECC 签名） |
| `requires` | array | 是 | 依赖的 Runtime 接口列表 |
| `entry_point` | string | 是 | 入口脚本 |
| `actions` | array | 是 | 支持的操作 |
| `description` | string | 是 | 技能描述 |

### 2.3 integrity vs trust（关键区分）

v3.0 严格区分两个概念：

```
integrity (hash)  → 文件有没有被修改（完整性校验）
trust (signature) → 是不是官方发布（信任签名）
```

| 字段 | 含义 | 算法 | 示例 |
|------|------|------|------|
| `integrity.algorithm` | 完整性算法 | sha256 | `"sha256"` |
| `integrity.hash` | 文件 SHA256 hash | sha256 | `"abc123..."` |
| `trust.issuer` | 签发者 | - | `"aos-runtime"` |
| `trust.algorithm` | 签名算法 | none / rsa-4096 / ecc-p256 | `"none"` (v2.x), `"rsa-4096"` (v3.0+) |

### 2.4 Bundle Hash（fingerprint 不写死在 manifest）

v3.0 关键变更：fingerprint 不再写死在 manifest.json 中。

Bundle 安装时由 Installer 计算 `hashes.json`，包含 `manifest.json` + `skill.ps1` 的独立 hash：

```json
{
  "manifest.json": "sha256:abc123...",
  "skill.ps1": "sha256:def456..."
}
```

验证链路：

```
Bundle Hash → Manifest → File Hash
```

类似 Docker image digest 机制。manifest 本身的完整性通过 bundle hash 校验。

### 2.5 compatibility 字段

```json
"compatibility": {
    "runtime": ">=2.2 <3.0",
    "protocol": "AISE-1.x",
    "skill_api": "1.0"
}
```

- `runtime`: 兼容的 Runtime 版本范围
- `protocol`: 兼容的协议版本范围
- `skill_api`: Skill API 版本

### 2.6 requires 字段

声明 Skill 依赖的 Runtime 接口。Runtime 在加载 Skill 前验证依赖是否满足：

```json
"requires": [
    "IContext>=1.0",
    "IEvent>=1.0"
]
```

可用接口：`IContext`, `IEvent`, `IIdentity`, `ILifecycle`, `IAdmission`, `ICapability`, `IAdapter`

如果依赖不满足，Runtime 拒绝加载该 Skill。

---

## Chapter 3 — State Machine & Skill Triggering

### 3.1 触发原则

Skill 由 Runtime State Machine 自动触发，不是 Agent 主动调用。

```
Agent → 产生状态变更 → State Machine → 匹配 Skill → 执行
```

Agent 的触发词（如 `AISE HANDOFF`）只是 Human Override，必须通过 Command Router：

```
Human Override → Runtime Command Router → Permission Check → State Machine → Skill
```

不能绕过 State Machine 直接调用 skill.ps1。

### 3.2 Known States（冻结列表）

| 状态 | 触发 Skill | 层级 |
|------|-----------|------|
| `project.created` | aise-bootstrap | 10-lifecycle |
| `project.attach` | aise-admission | 00-kernel |
| `agent.enter` | aise-admission, aise-inject | 00-kernel, 40-adapter |
| `agent.exit` | aise-handoff | 10-lifecycle |
| `handoff.request` | aise-handoff | 10-lifecycle |
| `release.triggered` | aise-archive | 10-lifecycle |
| `pre-commit` | aise-gitops | 20-governance |
| `pre-push` | aise-gitops | 20-governance |
| `protocol.check` | aise-fetch | 30-protocol |
| `environment.init` | aise-sync | 30-protocol |
| `environment.sync` | aise-sync | 30-protocol |
| `runtime.health` | aise-health | 50-maintenance |
| `runtime.update` | aise-update | 50-maintenance |

### 3.3 禁止的自我触发

Skill 不得监听自己产生的状态：

```
禁止: aise-admission 监听 admission.passed
禁止: aise-handoff 监听 handoff.completed
禁止: 任何 Skill 监听 action.completed / action.failed
```

State Machine 在触发时检测循环依赖，拒绝执行。

### 3.4 触发链路示例

```
Agent 进入项目
    │
    ├── State: agent.enter
    │       │
    │       └── aise-admission.validate()
    │               │
    │               ├── 通过 → State: admission.passed
    │               │              │
    │               │              └── aise-inject.inject()
    │               │
    │               └── 不通过 → 拒绝进入
    │
Agent 退出项目
    │
    ├── State: agent.exit
            │
            └── aise-handoff.export()
                    │
                    └── Emit event: agent.exit
```

---

## Chapter 4 — Skill Contract

### 4.1 统一入口签名

所有 Skill 必须遵循统一的入口签名：

```powershell
param(
    [Parameter(Mandatory=$true)]
    [string]$Action,

    [Parameter(Mandatory=$false)]
    [string]$ProjectPath,

    [Parameter(Mandatory=$false)]
    [hashtable]$Context = @{}
)
```

### 4.2 Runtime Interface 调用规范

Skill 必须通过以下 Helper 函数调用 Runtime 接口，不得直接读写 Runtime 内部状态：

```
Invoke-IEvent       → IEvent (eventbus.ps1)
Invoke-IContext     → IContext (context.ps1)
Invoke-IIdentity    → IIdentity (identity.ps1)
Invoke-ILifecycle   → ILifecycle (lifecycle.ps1)
Invoke-ICapability  → ICapability (capability.ps1)
```

Git 等外部工具通过 ICapability 接口调用，Skill 不直接执行 git/svn 等外部命令。

### 4.3 返回值约定

- 成功：返回 hashtable 或 `$true`，exit code 0
- 失败：返回 `$false` 或错误信息，exit code 1
- 事件：通过 IEvent 发射，不通过返回值传递

### 4.4 错误处理（Degraded Mode）

Skill 必须优雅处理 Runtime 接口不可用的情况，但**不得静默继续**：

```
Runtime 接口不可用
    → 降级为最小功能模式
    → 发射 IEvent: skill.degraded 警告事件
    → 记录到 Audit Log
    → 标记 snapshot 为 degraded = true
```

生产环境禁止静默丢弃上下文。

---

## Chapter 5 — Skill Lifecycle

### 5.1 生命周期状态

```
[defined] → [installed] → [active] → [deprecated] → [removed]
```

| 状态 | 含义 |
|------|------|
| `defined` | 在 aos-protocol-factory 中定义 |
| `installed` | 已安装到 CENTRE_HOME，integrity 已验证 |
| `active` | 当前激活，可被 State Machine 触发 |
| `deprecated` | 标记为废弃，触发时产生警告 |
| `removed` | 已从系统移除 |

### 5.2 安装流程

```
1. Installer 分发 Skill Bundle 到 CENTRE_HOME/system-skills/
2. 验证 bundle hash（hashes.json → manifest.json + skill.ps1）
3. 验证 compatibility 兼容性
4. 验证 requires 依赖满足
5. 写入 manifest.installed_at
6. 注册到 Skill Registry
7. Skill 标记为 active
```

### 5.3 卸载流程

```
1. 标记为 deprecated
2. 等待所有依赖 Skill 更新
3. 移除 Skill 文件
4. 从 Registry 注销
```

---

## Chapter 6 — Skill Registry

### 6.1 Registry 结构

```json
{
  "registry_version": "2.0",
  "updated_at": "2026-07-17T00:00:00Z",
  "skills": {
    "aise-admission": {
      "path": "00-kernel/aise-admission",
      "state": "active",
      "installed_version": "1.0.0",
      "installed_at": "2026-07-17T00:00:00Z",
      "triggers": ["agent.enter", "project.attach"]
    }
  }
}
```

### 6.2 Registry 加载顺序

Runtime 启动时按以下顺序加载 Skill：

```
1. 00-kernel/       → Kernel Security
2. 10-lifecycle/    → Lifecycle
3. 20-governance/   → Governance
4. 30-protocol/     → Protocol
5. 50-maintenance/  → System Maintenance
6. 40-adapter/      → Adapter（最后加载，依赖其他层）
```

---

## Chapter 7 — Adapter Registry

### 7.1 Adapter Registry 结构

Skill 不硬编码 Agent 路径。Agent 类型与入口路径的映射由 `registry/adapters.json` 提供：

```json
{
  "registry_version": "1.0",
  "adapters": {
    "trae": {
      "entry": ".trae/rules/aise.md",
      "detect": [".trae", "%USERPROFILE%/.trae"],
      "adapter": "runtime/adapters/trae/adapter.ps1"
    },
    "workbuddy": {
      "entry": ".workbuddy/rules/aise.md",
      "detect": ["%USERPROFILE%/.workbuddy/config.json"],
      "adapter": "runtime/adapters/workbuddy/adapter.ps1"
    }
  }
}
```

### 7.2 一个 Runtime Skill，多个 Adapter Renderer

不是三个 skill（trae-skill, workbuddy-skill, claude-skill），而是：

```
一个 Runtime Skill（aise-inject）
    +
多个 Adapter Renderer（trae / workbuddy / claude / cursor / vscode / workbench）
```

---

## Chapter 8 — Trust & Verification

### 8.1 信任链

```
aos-protocol-factory  → 定义 Skill 规格
        │
        v
aos-runtime           → 实现 Skill，签发 bundle hash
        │
        v
CENTRE_HOME           → 安装 Skill，验证 bundle hash
        │
        v
Runtime               → 加载 Skill，验证 manifest
```

### 8.2 验证层次（v3.0）

```
Level 1: integrity 完整性验证（SHA256 hash）
Level 2: compatibility 兼容性验证
Level 3: requires 依赖验证
Level 4: trust.issuer 信任链验证
Level 5: trust.signature 签名验证（v3.0+，RSA/ECC）
```

### 8.3 篡改检测

如果 Skill 文件的 SHA256 与 manifest.integrity.hash 不匹配：

```
1. 标记 Skill 为 compromised
2. 拒绝加载
3. 发射 IEvent: skill.compromised
4. 通知 Runtime 管理员
```

---

## Chapter 9 — Upgrade & Compatibility

### 9.1 版本兼容性矩阵

```
Runtime 版本     → 兼容的 Skill API 版本
─────────────────────────────────────
2.2.x            → Skill API 1.0
2.5.x            → Skill API 1.x
3.0.x            → Skill API 2.0 (可能)
```

### 9.2 升级策略

```
Skill 升级:
  1. 新版本 Skill 安装到 CENTRE_HOME
  2. 旧版本 Skill 标记为 deprecated
  3. State Machine 切换使用新版本
  4. 验证无 Skill 依赖旧版本
  5. 移除旧版本

Runtime 升级:
  1. 检查所有 Skill 的 compatibility.runtime
  2. 不兼容的 Skill 标记为 inactive
  3. 升级 Runtime
  4. 重新验证 Skill 兼容性
  5. 兼容的 Skill 恢复 active
```

---

## Chapter 10 — Skill Runtime Components

### 10.1 Runtime 组件

v3.0 引入完整的 Skill Runtime 组件：

| 组件 | 文件 | 职责 |
|------|------|------|
| Loader | `runtime/skill/loader.ps1` | 从 Registry 加载 Skill、按序加载、验证 manifest |
| Validator | `runtime/skill/validator.ps1` | 6 层验证：fields, compatibility, dependencies, trust, triggers, integrity |
| Executor | `runtime/skill/executor.ps1` | State Machine 驱动，State→Skill 匹配，Skill Action 执行 |
| Installer | `runtime/commands/install-skill.ps1` | 安装 Skill Bundle，验证 bundle hash，注册到 Registry |

### 10.2 Executor State→Action 映射

| State | Default Action |
|-------|---------------|
| `agent.enter` | validate |
| `agent.exit` | export |
| `project.created` | init |
| `project.attach` | validate |
| `handoff.request` | export |
| `release.triggered` | release |
| `pre-commit` | audit |
| `pre-push` | audit |
| `protocol.check` | check |
| `environment.init` | check |
| `environment.sync` | sync |
| `runtime.health` | check |
| `runtime.update` | check |

---

## Chapter 11 — Installer Integration

### 11.1 Installer 职责

AOS Installer 负责 Skill 的分发和安装：

1. 验证 Skill Bundle 来源（aos-protocol-factory）
2. 验证 bundle hash（hashes.json）
3. 解压到 CENTRE_HOME/system-skills/
4. 注册到 Skill Registry
5. 触发 `runtime.update` 状态

### 11.2 Skill Bundle 格式

```
aise-<name>-v<version>.bundle.zip
    ├── hashes.json       ← bundle 内各文件 hash
    ├── manifest.json
    ├── skill.ps1
    └── schema.json（可选）
```

### 11.3 Bundle Hash 验证

```
1. 解压 Bundle
2. 读取 hashes.json
3. 验证 manifest.json hash
4. 验证 skill.ps1 hash
5. 任一不匹配 → 拒绝安装
```

---

## Chapter 12 — Non-Scope

v3.0.0frozen 明确排除：

| 排除项 | 原因 |
|--------|------|
| 用户自定义 Skill | 仅 System Skill |
| Skill 市场/商店 | 非 v3.0 范围 |
| Skill 热加载 | 需要重启 Runtime |
| Skill 间通信 | 通过 IEvent，不直接调用 |
| Skill 沙箱 | 信任 System Skill |
| 加密签名（RSA/ECC） | v3.0+ 已预留字段，实现待后续版本 |

---

## 附录 A — 三层宪法关系

| 文档 | 位置 | 职责 |
|------|------|------|
| AISE Protocol Constitution | `aos-protocol-factory/Protocols/CONSTITUTION.md` | 定义 AISE 工程协议 |
| CENTRE Gateway Runtime Constitution | `aos-runtime/constitution/CONSTITUTION.md` | 定义 Runtime 身份与边界 |
| AOS System Skill Architecture Specification | `aos-runtime/constitution/SKILL_ARCHITECTURE.md` | 定义 System Skill 架构 |

三者关系：

```
AISE Protocol Constitution  → 定义"Agent 如何工作"
CENTRE Runtime Constitution → 定义"Runtime 如何执行协议"
AOS System Skill Spec       → 定义"Runtime 如何扩展系统能力"
```

---

## 附录 B — 三层仓库关系

```
aos-protocol-factory（宪法仓库）
    │  定义 Protocol / Contract / Schema / Policy / Skill Spec
    │  不运行
    │
    v
aos-runtime / CENTRE Gateway（操作系统内核）
    │  执行 Runtime / Interface / State Machine / Skill Runtime / Adapter / Installer / Registry
    │  运行
    │
    v
AgentWorkbench（AOS Desktop Shell）
    │  提供 UI / Project View / Agent Management / Human Interaction
    │  不是 Runtime
```

---

## 附录 C — 修订记录

| 版本 | 日期 | 修订内容 |
|------|------|---------|
| 1.0.0frozen | 2026-07-17 | 初版冻结：定义 System Skill 架构、Manifest Schema、State Machine、5 层分类、Trust 链、Registry、兼容性矩阵 |
| 3.0.0frozen | 2026-07-17 | 重大升级：拆分 integrity(hash) 与 trust(signature)，fingerprint 移入 bundle hash；目录重组为编号前缀（00-kernel ~ 50-maintenance）；引入 Adapter Registry、Command Router、Skill Runtime 组件（loader/validator/executor/installer）；Git 改为 ICapability 外部调用；Degraded Mode 增加 Event 警告 |