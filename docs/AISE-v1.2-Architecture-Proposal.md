# AISE v1.2 Architecture Proposal

> **Version**: v1.2.0-proposal  
> **Status**: Design Proposal — 不实施、不修改代码、不创建 Commit  
> **基线**: AISE v1.1.2（aise-standard v1.1.0-rc.2 + agent-governance v1.4.2）已冻结  
> **日期**: 2026-07-15  
> **分析范围**: `E:\Development\ACP+AISE`（含 `aise-standard`、`agent-governance` 及已注入项目模板）

---

## 0. Executive Summary: AISE v1.2 = Protocol Freeze

> **战略边界（Strategic Boundary）**: AISE v1.2 不是一次全面重构，而是一次 **Protocol Freeze**。  
> 本提案在 v1.1.2 稳定基线之上，仅冻结三套跨项目协议，其余能力留给 AOS / Agent Gateway Runtime 演进。

### 0.1 与 AOS / Gateway / Workbench 的关系

> 参见 **ADR-001: AISE Protocol Layer Separation**、**ADR-003: AOS / Workbench / Gateway Layer Separation**。

```text
ACP  (Agent Communication Protocol)
        ↓  管“Agent 之间怎么说话”
AISE (Agent Infrastructure Protocol)
        ↓  管“Agent 怎么施工”
Agent Workbench
        ↓  人机交互 + 项目管理 + 可视化验证
AOS Runtime
        ↓  Agent 运行环境、Session、Capability、Skill Execution
Agent Gateway
        ↓  MCP、External API、Model Router、Remote Agent、Service Discovery
Real Agent OS
```

- **AISE 从“系统”下降为“协议层”**：不再拥有 Registry、Gateway、MCP Server、Memory Service 等运行时实现，只规定项目入口、上下文结构、启动顺序。→ ADR-001
- **Agent Workbench 是 AISE 的第一个官方产品化验证平台**：任何 Provider、Tool、Skill、Workflow、Memory / Knowledge / Agent Identity 必须先在 Workbench 中验证，再决定是否进入 AOS Runtime。→ ADR-003
- **Registry / Gateway / Memory Index 最终进入 AOS Runtime / Agent Gateway**：v1.2 阶段 agent-governance 仓库仍可保留 `repository-identities.json` 作为过渡。→ ADR-001、ADR-003

### 0.2 v1.2 只冻结的三件事

| # | 协议 | 为什么必须冻结 |
|---|------|---------------|
| 1 | **`.agent-entry.json` 标准** | 任何 Agent、任何 IDE、任何 Gateway 进入项目的唯一入口契约 |
| 2 | **`.project/context` 标准** | Project Context 跨 Agent / 跨平台可迁移的 JSON/Event 骨架 |
| 3 | **Bootstrap Protocol v2** | Agent 启动读取顺序：Workspace → Project → Git Reality → Handoff → Project Goals → Governance |

### 0.3 v1.2 明确不做的四件事

| 能力 | v1.2 定位 | 负责方 |
|------|----------|--------|
| Registry（Project Registry / Identity Service） | 不实现 | AOS Runtime / Agent Gateway |
| Gateway（Capability Router / Policy Engine） | 不实现 | AOS Runtime / Agent Gateway |
| MCP Server / Client 运行时 | 不实现 | AOS Runtime / all_mcp_api |
| Memory Service / Memory Index | 不实现 | AOS Runtime / Agent Workbench |

> **结论**: v1.2 完成后，AISE 成为可嵌入任何 Agent Runtime 的轻量协议层；AOS 成为协议的消费方与扩展方。

### 0.4 Architecture Decision Records（已冻结）

v1.2 不是探索性提案，而是边界冻结。以下 ADR 定义最终决策：

| ADR | 标题 | 状态 | 解决的核心问题 |
|-----|------|------|----------------|
| **ADR-001** | AISE Protocol Layer Separation | Accepted — Frozen | AISE 是协议还是框架？ |
| **ADR-002** | Project Context Event Sourcing Model | Accepted — Frozen | Project Context 如何成为持久资产？ |
| **ADR-003** | AOS / Workbench / Gateway Layer Separation | Accepted — Frozen | Workbench、AOS、Gateway 边界在哪？ |
| **ADR-004** | AISE Distribution Strategy | Accepted — Transitional | AISE 如何进入项目？Hybrid → Protocol Only |

> **说明**: ADR-004 为 Transitional，因为 v1.2 采用 Hybrid Runtime，v2.0 才进入 Protocol Only。其余 ADR 为 Frozen，v1.2 起不可变更。

详细 ADR 内容见 [第 11 章](#11-architecture-decision-recordsadr)。

---

## 1. Current Problems

基于对 `aise-standard`、`agent-governance` 及 `aise-template` 的盘点，v1.1.2 稳定可用，但存在以下结构性问题，应在 v1.2 中通过职责拆分与模块化解决。

### P1: Workspace 与 Project 未区分

当前 Agent 进入 `E:\Development\ACP+AISE` 时，Bootstrap 直接检测当前目录是否存在 `AISE/SYSTEM.md`，导致输出 `AISE Detected: NO`。原因在于该目录是 **Workspace**（工作空间），包含多个项目（`aise-standard`、`agent-governance`），而非单一 Project。

- **影响**: Agent 无法在 Workspace 级别完成统一的入口治理，必须依赖用户手动进入子目录。
- **来源**: `aise-standard/Agent-Entry/Bootstrap.md` 的 Step 1 仅检测 `AISE/SYSTEM.md`。

### P2: AISE 标准自身结构 与 注入模板结构 不一致

- `aise-standard` 仓库将 `SYSTEM.md`、`VERSION`、`Registry/`、`Contracts/` 等直接置于仓库根目录。
- `agent-governance/aise-template` 将这些内容置于 `AISE/` 子目录下，并复制 `AGENTS.md`、`CLAUDE.md`、`.trae/rules/aise.md` 到项目根目录。

**结果**: 标准仓库本身并不是按“被注入项目”的结构组织，导致 aise-standard 既是“宪法源”，又是“扁平化的特殊项目”，其他项目无法直接以 aise-standard 为样板。

### P3: Registry 重复与双写

`agent-governance/.agent-governance/` 下存在两套注册表：

- `registry/projects.json`：含 `local_path`、`language`、`framework`、`current_version`、`last_known_commit` 等。
- `repository-identities.json`：含 `github_repo`、`origin`、`default_branch`、`protected_branches`、`branch_strategy` 等。

两者都包含 `project_name`、`default_branch`、`protected_branches`、`development_branch`、`origin`、`description`、`governance_entry`。v1.4.2 甚至引入“双写 + `registry-sync.ps1`”来保持一致，这是典型的职责混淆。

- **影响**: 单用户单项目场景下维护两份数据，易产生 drift；多项目/多 Agent 场景下无法扩展。
- **来源**: `agent-governance/PROJECT_BLUEPRINT.md` v1.4.2 变更记录。

### P4: Agent 入口文件重复

`AGENTS.md`、`CLAUDE.md`、`.trae/rules/aise.md` 三份文件内容几乎相同（同样的 WARNING 与读取顺序）。此外 `AISE/SYSTEM.md` 与这些入口文件也存在内容重叠。

- **影响**: 更新入口协议时需要同步修改多份文件，违反 DRY。
- **来源**: `agent-governance/aise-template/AGENTS.md`、`CLAUDE.md`、`.trae/rules/aise.md`。

### P5: Bootstrap 加载顺序：治理规则先于项目现实

当前 Bootstrap Protocol v1 的顺序为：

```
AGENTS.md / CLAUDE.md / Trae Rule
→ AISE/Agent-Entry/Bootstrap.md
→ AISE/SYSTEM.md
→ PROJECT_BLUEPRINT.md
→ CHANGELOG.md
→ .project/
→ AISE Verify
```

Agent 在尚未确认“当前 git 分支是否干净、当前项目目标是什么、是否有未完成的 Handoff”之前，先加载完整 AISE 治理。这导致：

- Agent 先学习“通用规则”，再了解“当前任务”，与 Mission 驱动的原则相悖。
- Git 现实状态（dirty/clean、branch、origin、HEAD）在 Bootstrap 末尾才检查。

### P6: Handoff 位置不统一

- `AISE/Contracts/handoff-protocol.md` 规定 Handoff 位于 `<project>/.handoff/HANDOFF.md`。
- `.agent-entry.json` 的 `handoff.location` 指向 `agent-governance/.agent-governance/handoff/{project}`。
- `agent_exit.ps1` 实际写入 `agent-governance/.agent-governance/handoff/{project}/latest.json`。

项目本地 `.handoff/` 与治理仓库 Handoff 历史并存，但缺乏统一索引，存在两份 Handoff 数据冲突的风险。

### P7: Project Context 目录结构碎片化

`aise-init.ps1` 创建 `.project/architecture`、`contracts`、`decisions`、`prompts`、`feedback`、`roadmap`、`standards`，而 `Policies/memory-ownership.md` 定义的统一结构为 `.project/memory/{knowledge,decisions,architecture,patterns,glossary}` + `journal/` + `mission/` + `handoff/` + `audit/` + `releases/`。

- **影响**: 初始化脚本与治理策略不一致，项目上下文散落在多处。
- **来源**: `agent-governance/scripts/aise-init.ps1`、`aise-standard/Policies/memory-ownership.md`。

### P8: 机器相关数据与治理数据混放

`workspace-bindings.json` 被 `.gitignore` 排除（含本地路径），但 `workspace-registry.json` 却被提交并引用 `workspace-bindings.json`。该组合索引在全新 clone 下失效，因为 `workspace-bindings.json` 不存在。

- **影响**: `workspace-registry.json` 作为“Primary lookup for Agents”依赖缺失文件，无法独立工作。
- **来源**: `agent-governance/.agent-governance/workspace-registry.json`、`agent-governance/.gitignore`。

---

## 2. Responsibility Separation

### 2.1 五域定义

| 域 | 职责 | 核心问题 |
|----|------|---------|
| **Project Domain** | 项目本身：源代码、测试、文档、配置、架构、需求、蓝图、版本历史 | “项目是什么？” |
| **Project Context** | 项目当前状态：记忆、交接、日志、决策、任务、进度、边界 | “项目现在做到哪里？” |
| **AISE Runtime** | Agent 如何进入项目：身份识别、启动协议、技能绑定、策略、运行时配置、Git 治理 | “Agent 如何进入项目？” |
| **Central Governance** | 跨项目统一治理：仓库身份、Agent 身份契约、生命周期规则、跨项目 Handoff 索引 | “谁被允许做什么？” |
| **Git Governance** | Git 出入口控制：Hooks、Commands、Gate Policies、审计触发 | “Git 操作如何被保护？” |

### 2.2 文件职责清单

#### aise-standard 仓库（宪法源）

| 路径 | 当前职责 | 建议归属域 |
|------|---------|-----------|
| `SYSTEM.md` | Agent 系统提示词、九层架构、安全元规则、触发词路由 | AISE Runtime |
| `VERSION` | AISE 版本锁定 | AISE Runtime |
| `Contracts/` | 工程、元数据、仓库、交接四份契约 | AISE Runtime |
| `Policies/` | 安全、Git、回滚、豁免、审计、Mission、ADR、身份、升级、Memory 所有权 | AISE Runtime |
| `Skills/` | Skill 定义（archive/handoff/gitops/review/publish/sync/fetch/verify） | AISE Runtime |
| `Registry/` | 版本、路由、技能、合规、Git 治理能力注册表 | AISE Runtime |
| `Agent-Entry/` | Bootstrap、COMPLIANCE、AGENTS.md、CLAUDE.md、`.trae/rules/aise.md` | AISE Runtime |
| `Git-Governance/` | git-gate、hooks、commands、policies.json | Git Governance |
| `Templates/` | BLUEPRINT、CHANGELOG、HANDOFF、REPORT 模板 | AISE Runtime / Project Construction |
| `Migrations/` | 版本迁移脚本 | AISE Runtime |
| `.agent-entry.json` | 指向 agent-governance | Project Context（指针） |

> **问题**: `aise-standard` 根目录结构扁平化，与其注入模板 `AISE/` 子目录不一致。v1.2 应明确 `aise-standard` 是“中央宪法源”，不应再扮演“被注入项目”。

#### agent-governance 仓库（中央治理）

| 路径 | 当前职责 | 建议归属域 |
|------|---------|-----------|
| `.agent-governance/agent-contract.json` | Agent 生命周期（enter/work/exit）规则 | Central Governance |
| `.agent-governance/agent-identity-contract.json` | Agent 类型、权限、当前 Agent 身份 | Central Governance |
| `.agent-governance/repository-identities.json` | 每个仓库的不可变身份（GitHub 级） | Central Governance（Primary Data） |
| `.agent-governance/registry/projects.json` | 项目索引，含本地路径、语言、版本等 | Central Governance（Cache/Index） |
| `.agent-governance/workspace-registry.json` | identity + binding 组合索引 | Workspace（应迁移） |
| `.agent-governance/workspace-bindings.json` | 机器相关的本地路径与分支 | Workspace（Primary Data，但不提交） |
| `.agent-governance/git-policy.json` | Git 操作规则 | Git Governance / Central Governance |
| `.agent-governance/handoff/` | 跨项目 Handoff 历史 | Central Governance（历史归档） |
| `.agent-governance/memory/` | 跨项目 Memory 索引 | Central Governance（索引） |
| `.agent-governance/journal/` | Agent 活动日志 | Central Governance / Workspace |
| `scripts/agent_bootstrap.ps1` | 入口门 | AISE Runtime |
| `scripts/agent_exit.ps1` | 出口门 | AISE Runtime |
| `scripts/aise-init.ps1` | 项目初始化注入 | AISE Runtime |
| `aise-template/` | 注入模板 | AISE Runtime / Project Construction |

#### 已注入项目（如 `aise-template` 所生成）

| 路径 | 当前职责 | 建议归属域 |
|------|---------|-----------|
| `src/`, `tests/`, `docs/`, `config/` | 业务代码与文档 | Project Domain |
| `PROJECT_BLUEPRINT.md` | 项目架构全景 | Project Domain |
| `CHANGELOG.md` | 版本历史 | Project Domain |
| `README.md`, `.gitignore` | 项目说明与忽略规则 | Project Domain |
| `AISE/` | 完整 AISE 运行时拷贝 | AISE Runtime（建议改为轻量指针） |
| `AGENTS.md`, `CLAUDE.md`, `.trae/rules/aise.md` | Agent 入口提示 | AISE Runtime（最小入口文件） |
| `.agent-entry.json` | 指向 Central Governance | Project Context（指针） |
| `.project/` | Mission、Memory、Decisions、Audit、Handoff | Project Context |
| `.agent/` | identity.json, capability.json, session.json | Project Context（Agent 会话） |
| `.handoff/` | 项目本地 Handoff | Project Context（本地缓存） |
| `.sync/` | 同步状态 | Project Context |

---

## 3. Project Context Model

### 3.1 Project vs Project Context

| 维度 | Project Domain | Project Context |
|------|---------------|-----------------|
| 问题 | 项目是什么？ | 项目现在做到哪里？ |
| 内容 | 源代码、架构、文档、依赖、Blueprint、Changelog | Memory、Handoff、Journal、Decisions、Tasks、Mission、Progress |
| 生命周期 | 随版本演进，相对稳定 | 随每次 Agent 会话变化，高频更新 |
| 所有权 | 项目资产，用户与 AI 共同维护 | 项目资产，但 Agent 产生大量临时/过程数据 |
| 存储位置 | 仓库根目录业务文件 | `.project/` + `.handoff/` + `.agent/`（会话级） |

### 3.2 当前职责混淆

当前问题：

1. `PROJECT_BLUEPRINT.md` 同时承担“项目目标”和“版本历史”职责，而后者应由 `CHANGELOG.md` 承担。
2. `.project/` 目录在 `aise-init.ps1` 与 `memory-ownership.md` 中定义不一致。
3. `.agent/` 既存 `identity.json/capability.json`（Agent 会话），又被误用于 Memory（已被 Policy 禁止）。
4. Handoff 既在项目本地 `.handoff/`，又在 `agent-governance/.agent-governance/handoff/`，缺乏统一索引。

### 3.3 v1.2 推荐的 `.project/` 结构

> 参见 **ADR-002: Project Context Event Sourcing Model**。

```text
.project/
├── context.json              # 项目上下文顶层索引（新增）
├── context/                  # Project Context 主域（JSON/Event 层）
│   ├── timeline.jsonl        # Canonical Event Source（规范事件源）
│   ├── mission.json          # Mission Boundary 物化视图（Derived）
│   ├── state.json            # 当前任务状态物化视图（Derived）
│   ├── decisions/            # 项目级决策（非 ADR）
│   ├── memory/               # 项目记忆
│   │   ├── index.json
│   │   ├── knowledge/        # 领域知识
│   │   ├── architecture/     # 架构文档
│   │   ├── patterns/         # 代码模式
│   │   └── glossary/         # 术语表
│   └── handoff/              # Handoff 物化视图
│       ├── latest.json       # 由 timeline 聚合生成（Derived Snapshot）
│       └── latest.md         # 展示层（Derived Presentation）
├── mission/                  # 治理策略定义的 Mission Boundary（保留）
│   ├── scope.json
│   └── constraints.json
├── journal/                  # Agent 活动日志
│   └── 2026/
│       └── 07/
│           └── 15.json
├── decisions/                # ADR 重大架构决策
│   └── ADR-001-*.md
├── audit/                    # 审计日志
│   ├── gate-events.jsonl
│   ├── agent-events.jsonl
│   └── authorization-events.jsonl
└── handoff/                  # v1.1.2 兼容：逐步迁移到 .project/context/handoff/
    ├── HANDOFF.md
    └── history/
```

#### context.json 示例

```json
{
  "schema_version": "1.2.0",
  "project_id": "aise-standard",
  "last_updated": "2026-07-15T12:00:00+08:00",
  "pointers": {
    "blueprint": "PROJECT_BLUEPRINT.md",
    "changelog": "CHANGELOG.md",
    "mission": ".project/context/mission.json",
    "state": ".project/context/state.json",
    "timeline": ".project/context/timeline.jsonl",
    "memory_index": ".project/context/memory/index.json",
    "handoff_latest": ".project/context/handoff/latest.json",
    "handoff_presentation": ".project/context/handoff/latest.md",
    "audit_dir": ".project/audit/",
    "agent_identity": ".agent/identity.json"
  },
  "aise": {
    "version": "1.2.0-proposal",
    "provider": "agent-gateway",
    "manifest": "AISE/manifest.json"
  }
}
```

### 3.4 关键设计点

> 参见 **ADR-002: Project Context Event Sourcing Model**。

- **Project Context 属于项目**: 必须随 Git 仓库移动，其他 Agent 克隆后可读。
- **Source of Truth Hierarchy**: Git History（Level 0）→ `timeline.jsonl`（Level 1, Canonical Event Source）→ Materialized State（Level 2）→ Presentation（Level 3）。
- **Handoff 物化视图在项目本地**: `.project/context/timeline.jsonl` 为 Canonical Event Source；`.project/context/handoff/latest.json` 为 Derived Snapshot，`.project/context/handoff/latest.md` 为 Derived Presentation；`agent-governance` 仅保留索引与历史归档，避免 Handoff 双写。
- **Mission Boundary 是上下文的一部分**: 在 Agent 工作前加载，用于约束后续所有操作。
- **JSON/Event 优先，Markdown 为展示层**: Project Context 的 Canonical Event Source 由 `timeline.jsonl` 承载，Snapshot 与 Markdown 均为 Derived。

---

## 4. Workspace Model

### 4.1 Workspace != Project

| 维度 | Workspace（工作空间） | Project（项目） |
|------|---------------------|----------------|
| 定义 | 用户机器上的一个目录，可包含多个项目 + 治理仓库克隆 | 一个 Git 仓库，含业务代码与上下文 |
| 范围 | 机器级 | 仓库级 |
| 数据 | 本地路径、活跃分支、最后提交、Agent 会话 | 代码、文档、Context、AISE 运行时 |
| 是否提交 | 否（`.agent-workspace.json` 应 gitignored） | 是（除 `.agent/session.json` 等临时文件） |
| 典型路径 | `E:\Development\ACP+AISE` | `E:\Development\ACP+AISE\aise-standard` |

### 4.2 `.agent-workspace.json` 设计

应新增 Workspace 级入口文件，位于 Workspace 根目录（如 `E:\Development\ACP+AISE\.agent-workspace.json`），并在 `.gitignore` 中排除。

```json
{
  "schema_version": "1.2.0",
  "workspace_id": "ws-acp-aise-local",
  "machine_fingerprint": "sha256-of-machine-id",
  "governance": {
    "provider": "github",
    "repository": "lovingxiong-dot/agent-governance",
    "clone_url": "git@github.com:lovingxiong-dot/agent-governance.git",
    "local_path": "E:\\Development\\ACP+AISE\\agent-governance"
  },
  "aise_standard": {
    "repository": "lovingxiong-dot/aise-standard",
    "clone_url": "git@github.com:lovingxiong-dot/aise-standard.git",
    "local_path": "E:\\Development\\ACP+AISE\\aise-standard"
  },
  "projects": {
    "aise-standard": {
      "local_path": "E:\\Development\\ACP+AISE\\aise-standard",
      "origin": "git@github.com:lovingxiong-dot/aise-standard.git",
      "active_branch": "main",
      "last_known_commit": "208700bed6256e21d53f2329ec37e4abde8a1b37",
      "handoff_index": "agent-governance/.agent-governance/handoff/aise-standard/latest.json"
    },
    "agent-governance": {
      "local_path": "E:\\Development\\ACP+AISE\\agent-governance",
      "origin": "git@github.com:lovingxiong-dot/agent-governance.git",
      "active_branch": "main",
      "last_known_commit": "...",
      "handoff_index": "agent-governance/.agent-governance/handoff/agent-governance/latest.json"
    }
  },
  "current_agent": {
    "agent_id": "trae-work-beneficial-wallaby",
    "agent_type": "admin",
    "authorized_by": "lovingxiong-dot"
  }
}
```

#### Workspace 职责

- 发现本机所有 AISE 项目（扫描 `.agent-entry.json`）。
- 绑定 `repository-identities.json` 中的不可变身份到本地路径。
- 缓存 Handoff 索引指针。
- 管理 Agent 会话身份（`current_agent`）。
- 不存储任何项目业务数据。

#### Project 职责

- 存储业务代码、文档、Blueprint、Changelog。
- 存储 Project Context（`.project/`、`.handoff/`、`.agent/`）。
- 包含 AISE 运行时声明（`.agent-entry.json` + 最小入口文件 + `AISE/manifest.json`）。
- 作为 Git 资产被版本控制。

### 4.3 Agent 启动流程 v2

```text
Workspace Bootstrap
    ↓
读取 .agent-workspace.json（或自动发现）
    ↓
确认 Central Governance 本地克隆可用
    ↓
Project Discovery：扫描 Workspace 下所有 .agent-entry.json
    ↓
Project Selection：
    ├─ 当前目录是 Project → 直接选中
    └─ 当前目录是 Workspace → 列出项目供用户选择
    ↓
Project Bootstrap
    ↓
Runtime Ready
```

---

## 5. Registry Redesign

### 5.1 当前 Registry 分析

#### `repository-identities.json`（当前）

- 优点: 明确声明“不可变 GitHub 级身份”。
- 内容: `project_name`、`github_repo`、`default_branch`、`protected_branches`、`development_branch`、`origin`、`description`、`tags`、`branch_strategy`、`archived_branches`、`governance_entry`。
- 归属: **Central Governance Primary Data**。

#### `registry/projects.json`（当前）

- 内容: 除上述身份字段外，还包含 `local_path`、`language`、`framework`、`type`、`current_version`、`owner`、`has_local_clone`、`last_known_commit`、`last_updated`、`injected_at`。
- 归属: 混合了 Central Governance 与 Workspace 数据。
- 问题: 与 `repository-identities.json` 大量重复，v1.4.2 需要“双写 + sync”维护。

#### `workspace-registry.json`（当前）

- 内容: 引用 `repository-identities.json` + `workspace-bindings.json` + handoff 路径。
- 问题: 依赖未提交的 `workspace-bindings.json`，作为“Primary lookup”无法独立工作。

### 5.2 是否需要双 Registry？

**结论**: 当前单用户单项目模型不需要两套重叠的 Registry。v1.2 应拆分为：

1. **不可变身份注册表**（Central Governance，提交）: 每个仓库是谁、在哪、分支策略是什么。
2. **工作空间绑定**（Workspace，不提交）: 本机哪个路径对应哪个仓库、当前分支、最后提交。
3. **跨项目索引**（Workspace/Central，可提交）: 由身份注册表 + 绑定表生成，属于 **Cache**。

未来多 Agent、多项目、多 Repo 场景下，上述拆分更加必要：

- 多 Workspace（多机器）共享同一 `repository-identities.json`。
- 每个 Workspace 有自己的 `workspace-bindings.json`。
- Agent 通过 `workspace_id` 区分不同机器状态。

### 5.3 v1.2 Registry Proposal

#### Central Governance（agent-governance，提交）

`repository-identities.json` —— **唯一事实源（Primary Data）**

```json
{
  "schema_version": "1.2.0",
  "description": "Immutable repository identities — Source of Truth.",
  "repositories": {
    "aise-standard": {
      "project_name": "aise-standard",
      "github_repo": "lovingxiong-dot/aise-standard",
      "origin": "git@github.com:lovingxiong-dot/aise-standard.git",
      "default_branch": "main",
      "protected_branches": ["main"],
      "development_branch": "main",
      "description": "Agent Software Engineering Protocol",
      "tags": ["standard"],
      "branch_strategy": { ... },
      "governance_entry": ".agent-entry.json"
    }
  }
}
```

仅含不可变身份与分支策略，不含 `local_path`、`last_known_commit` 等机器相关字段。

#### Per-Workspace（`.agent-workspace.json`，不提交）

`workspace-bindings.json` 的功能合并进 `.agent-workspace.json` 的 `projects` 段，不再单独存在。

#### Optional Cache（agent-governance，可提交/可忽略）

`projects.json` 保留为 **Cache/Index**，由 CI 或 `agent_exit.ps1` 根据 `repository-identities.json` + 各 Workspace 上报生成。字段仅包含：

- `project_id`
- `identity_ref`: `repository-identities.json#/repositories/{id}`
- `latest_version`（从项目 tag 或 Blueprint 抓取）
- `language/framework`（从项目元数据抓取）

明确标注为 **Derived Cache**，允许删除后重建。

### 5.4 Primary vs Cache 对照

| 数据 | Primary / Cache | 位置 | 是否提交 |
|------|----------------|------|---------|
| Repository Identity | Primary | `agent-governance/repository-identities.json` | 是 |
| Workspace Binding | Primary | `.agent-workspace.json#projects` | 否 |
| Project Context Event Source | Primary | `.project/context/timeline.jsonl` | 是 |
| Handoff Snapshot | Derived | `.project/context/handoff/latest.json` | 是 |
| Handoff Presentation | Derived | `.project/context/handoff/latest.md` | 是 |
| Handoff Index | Cache | `agent-governance/.agent-governance/handoff/index.json` | 是 |
| Projects Metadata Cache | Cache | `agent-governance/.agent-governance/registry/projects.json` | 是（可重建） |
| Workspace Registry | Cache | 移除，由 `.agent-workspace.json` 替代 | - |

---

## 6. Bootstrap Protocol v2

### 6.1 v1 的问题

- 先加载 AISE Runtime，再了解项目目标与 Git 现实。
- Handoff 加载顺序靠后，导致 Agent 无法在进入时立即获得“当前任务/阻塞点”。
- 未区分 Workspace 与 Project，进入 Workspace 直接失败。

### 6.2 推荐加载顺序原则

Agent 首先应理解：

1. **项目目标** — `PROJECT_BLUEPRINT.md`
2. **当前状态** — Handoff、Mission、Project Context
3. **Git 现实状态** — branch、origin、HEAD、dirty/clean
4. **治理规则** — AISE Runtime、Contracts、Policies

### 6.3 Bootstrap Protocol v2（14 步）

```text
Step 1:  Workspace Discovery
    ├── 检查当前目录是否为 Project（存在 .agent-entry.json）
    ├── 是 → 进入 Project Bootstrap
    └── 否 → 读取 .agent-workspace.json 进入 Workspace Bootstrap

Step 2:  Environment Snapshot
    ├── 操作系统、当前路径、Git 是否可用
    └── 输出 Workspace/Project 选择结果

Step 3:  Load .agent-entry.json
    ├── 解析 aise / governance / repository / handoff 指针
    └── 确定 Governance 仓库本地位置

Step 4:  Git Reality Check
    ├── git status --short
    ├── git branch --show-current
    ├── git rev-parse HEAD
    └── git remote get-url origin

Step 5:  Load Project Context — State
    ├── 读取 .project/context/state.json
    └── 识别当前任务 / Blocker / 未完成项

Step 6:  Load Project Context — Handoff
    ├── 读取 .project/context/handoff/latest.json
    └── 或从 governance handoff index 定位 latest.json

Step 7:  Load Project Context — Mission Boundary
    ├── .project/context/mission.json
    └── .project/mission/scope.json（v1.1.2 兼容）

Step 8:  Load Project Memory & Decisions
    ├── .project/context/memory/index.json
    ├── .project/context/decisions/
    ├── .project/decisions/ADR-*.md
    └── .project/audit/（可选读取最近事件）

Step 9:  Load Project Goals
    ├── PROJECT_BLUEPRINT.md
    ├── CHANGELOG.md
    └── README.md

Step 10: Resolve Governance
    ├── 通过 .agent-entry.json 定位 agent-governance
    ├── 读取 repository-identities.json 交叉验证
    └── 拉取/确认 Central Governance 最新

Step 11: Load Agent Identity
    ├── 读取 agent-governance/.agent-governance/agent-identity-contract.json
    ├── 确认当前 Agent 类型、权限、Git 身份
    ├── .agent/identity.json
    └── .agent/capability.json

Step 12: Load AISE Runtime
    ├── AISE/VERSION
    ├── AISE/manifest.json（v1.2 新增）
    ├── AISE/SYSTEM.md
    ├── AISE/Contracts/
    ├── AISE/Policies/
    ├── AISE/Skills/
    └── AISE/Registry/

Step 13: Load Git Governance
    ├── AISE/Git-Governance/git-gate.md
    ├── AISE/Git-Governance/policies.json
    └── 检查 .git/hooks/ 安装状态

Step 14: Compliance Check & Ready
    ├── 执行 AISE Verify
    └── 输出 Bootstrap Report
```

### 6.4 与 v1 的关键差异

| 步骤 | v1 | v2 |
|------|----|----|
| Workspace 识别 | 无 | Step 1-2 |
| 入口 | `AISE/SYSTEM.md` | `.agent-entry.json`（Step 3） |
| Git 现实 | 未在 Bootstrap 中 | Step 4（最前） |
| Project State | 无 | Step 5（新增） |
| Handoff | Markdown / 隐式加载 | JSON（Step 6，Project Goals 之前） |
| Mission Boundary | Step 5 | Step 7（紧接 State/Handoff） |
| Project Goals | Step 4 | Step 9（AISE 之前） |
| Governance/Identity | Step 3 | Step 10-11（了解项目之后） |
| AISE Runtime | Step 3 | Step 12（了解项目之后） |

---

## 7. AISE Protocol Distribution Model

> 参见 **ADR-004: AISE Distribution Strategy**。

### 7.1 当前模式：完整复制

每个项目通过 `aise-init.ps1` 复制完整 `AISE/` 目录（含 Contracts、Policies、Skills、Templates、Git-Governance）。

### 7.2 三种方案对比

#### Option A: 继续复制完整 AISE

| 维度 | 评估 |
|------|------|
| 优点 | 项目自包含；无外部依赖；离线可用；版本锁定简单 |
| 缺点 | 每个项目 duplication；升级需逐个 Migration；标准更新难同步；仓库体积大 |
| 适用场景 | 单项目、无网络、对标准稳定性要求极高的遗留项目 |

#### Option B: 中央 aise-standard + 项目轻量 Runtime

| 维度 | 评估 |
|------|------|
| 优点 | 单一事实源；升级只需更新中央仓库；项目体积小；无 duplication |
| 缺点 | 依赖中央仓库本地克隆；Agent 必须能解析路径；版本锁定需 manifest；离线不可用 |
| 适用场景 | 多项目、同 Workspace、频繁迭代 AISE 标准的场景 |

#### Option C: 混合模式（v1.2 Protocol Freeze 推荐）

项目保留：

- `.agent-entry.json` — 指向 Central Governance
- `AISE/VERSION` — 版本锁定
- `AISE/manifest.json` — 声明当前项目引用的 AISE 文件列表及哈希
- `AISE/Agent-Entry/` 中的最小入口文件（`AGENTS.md`、`CLAUDE.md`、`.trae/rules/aise.md`）— 因为 IDE 需要从项目根目录发现它们
- `AISE/Git-Governance/hooks/` — 从中央仓库安装到 `.git/hooks/`，但项目可保留一份本地清单

中央 `aise-standard` 提供：

- `SYSTEM.md`、完整 `Contracts/`、`Policies/`、`Skills/`、`Registry/`、`Templates/`、`Migrations/`、`Git-Governance/commands/`

Agent 启动时：

1. 读取项目 `AISE/manifest.json` 确认版本。
2. 通过 `.agent-workspace.json` 定位 `aise-standard` 本地克隆。
3. 加载中央文件；若本地克隆缺失，可自动克隆或降级到项目缓存。

| 维度 | 评估 |
|------|------|
| 优点 | 保留项目入口自发现能力；中央标准单点维护；版本锁定明确；可离线回退到本地缓存 |
| 缺点 | 实现复杂度高于 A；需要 Workspace 级别的 aise-standard 路径解析 |
| 适用场景 | **推荐作为 v1.2 默认模式** |

#### Option D: 纯协议指针（长期方向）

项目完全不保留 `AISE/` 目录，只保留：

```text
project/
├── .agent-entry.json      # 指向 aise-standard + agent-governance + runtime provider
├── .project/
├── .agent/
└── source code
```

`.agent-entry.json` 示例：

```json
{
  "schema_version": "2.0.0",
  "aise_version": "1.2.0",
  "provider": "agent-gateway",
  "runtime": "aise"
}
```

| 维度 | 评估 |
|------|------|
| 优点 | 项目零 AISE 代码；标准升级完全外部化；与 package.json 对 npm 的模型一致 |
| 缺点 | 强依赖 Gateway/Runtime 本地可用；需要统一运行时安装机制；旧 IDE 可能不支持 |
| 适用场景 | AOS Runtime 成熟后，作为 AISE v2.0 默认模式 |

### 7.3 推荐结论

> 参见 **ADR-004: AISE Distribution Strategy**。

**v1.2 Protocol Freeze 采用 Option C（混合模式）**，并明确：

- `aise-standard` 是唯一 Source of Truth。
- 项目不再拷贝完整 `AISE/`，仅保留入口指针 + manifest + IDE 入口文件。
- `agent-governance/scripts/aise-init.ps1` 改为生成轻量 runtime 并写入 manifest。
- 升级时通过 manifest diff 决定需要同步的文件。
- 长期演进方向为 **Option D（纯协议指针）**，由 AOS Runtime / Agent Gateway 在 v2.0 阶段消费，项目最终不保留任何 `AISE/` 目录。

### 7.4 `AISE/manifest.json` 示例

```json
{
  "schema_version": "1.2.0",
  "aise_version": "1.2.0-proposal",
  "source": "lovingxiong-dot/aise-standard",
  "files": {
    "SYSTEM.md": { "path": "SYSTEM.md", "hash": "sha256:..." },
    "Contracts/engineering-contract.md": { "path": "Contracts/engineering-contract.md", "hash": "sha256:..." },
    "Policies/security-policy.md": { "path": "Policies/security-policy.md", "hash": "sha256:..." },
    "Skills/archive/SKILL.md": { "path": "Skills/archive/SKILL.md", "hash": "sha256:..." },
    "Registry/version.json": { "path": "Registry/version.json", "hash": "sha256:..." }
  },
  "local_overrides": []
}
```

---

## 8. Migration Strategy

> **约束**: v1.1.2 保持冻结。v1.2 仅提交设计方案，不实施。

若未来实施 v1.2，建议按以下阶段推进。注意：v1.2 是 **Protocol Freeze**，不是全面重构，优先完成 Schema 与入口顺序冻结。

### Phase 0: Protocol Freeze — Schema 冻结

- 定义 `.agent-entry.json` v2.0.0 Schema。
- 定义 `.agent-workspace.json` v1.2.0 Schema。
- 定义 `.project/context.json` v1.2.0 Schema。
- 定义 `.project/context/state.json`、`.project/context/mission.json`、`.project/context/handoff/latest.json` Schema。
- 定义 `AISE/manifest.json` v1.2.0 Schema。
- 更新 `agent-governance/schema/handoff-schema.json` 以支持 Workspace/Project 指针。

### Phase 1: Workspace 层引入

- 在 `agent-governance` 新增 `.agent-workspace.json` 模板。
- `setup_agent_governance.ps1` 生成 Workspace 文件。
- `workspace-bindings.json` 功能合并进 `.agent-workspace.json`，原文件废弃。

### Phase 2: Registry 重构

- `repository-identities.json` 只保留不可变身份。
- `registry/projects.json` 改为 Derived Cache，标注可重建。
- 移除 `workspace-registry.json` 或改为 Workspace 级索引。

### Phase 3: AISE Runtime 轻量化

- 更新 `aise-template/`：移除完整 `AISE/` 拷贝，改为 `AISE/manifest.json` + 最小入口文件。
- `aise-init.ps1` 根据 manifest 从 `aise-standard` 拉取文件或生成轻量 skeleton。
- `aise-standard` 本身也使用 manifest 模式，统一结构。

### Phase 4: 入口/出口门升级

> 参见 **ADR-002: Project Context Event Sourcing Model**。

- `agent_bootstrap.ps1` 实现 Bootstrap Protocol v2：Workspace → Project → Git Reality → State → Handoff → Mission → Project Goals → Governance → AISE Runtime。
- `agent_exit.ps1` 追加事件到 `.project/context/timeline.jsonl`，并由 AOS Runtime 生成：
  - `.project/context/handoff/latest.json`（Derived Snapshot）
  - `.project/context/handoff/latest.md`（Derived Presentation）
  - `agent-governance/.agent-governance/handoff/{project}/latest.json`（历史归档 + 索引）

### Phase 5: v1.1.2 → v1.2 迁移脚本

> 参见 **ADR-002**、**ADR-004**。

- 扫描所有已注入项目。
- 将完整 `AISE/` 转换为 manifest + 最小入口文件（ADR-004）。
- 将 `.project/handoff/HANDOFF.md` 解析为事件写入 `.project/context/timeline.jsonl`，并生成 `.project/context/handoff/latest.json` + `latest.md`（ADR-002）。
- 将 `.project/mission/scope.json` 复制/迁移到 `.project/context/mission.json`。
- 生成 `.project/context/state.json` 与 `.project/context.json`。
- 将项目本地 Handoff 同步到 governance handoff index。
- 为每个 Workspace 生成 `.agent-workspace.json`。
- 更新 `AISE/VERSION` 到 1.2.0。

### 8.1 Project Construction Templates

> 参见 **ADR-002: Project Context Event Sourcing Model**、**ADR-004: AISE Distribution Strategy**。

结合 AgentWorkbench 的工程施工规范，v1.2 初始化时应根据项目类型生成不同骨架：

#### 通用生成项

无论哪种类型，创建项目时都应生成：

- 项目结构（按类型定制）
- `PROJECT_BLUEPRINT.md`（带元信息、定位、技术栈、目录结构）
- `CHANGELOG.md`
- `README.md`
- `.gitignore`
- `.project/context.json`
- `.project/context/mission.json`
- `.project/context/state.json`
- `.project/context/timeline.jsonl`
- `.project/context/memory/index.json`
- `.project/context/decisions/.gitkeep`
- `.project/context/handoff/.gitkeep`
- `.project/mission/scope.json` + `constraints.json`（v1.1.2 兼容）
- `.project/decisions/.gitkeep`
- `.project/audit/.gitkeep`
- `.project/handoff/.gitkeep`（v1.1.2 兼容）
- `.agent/identity.json` + `capability.json`
- `.agent-entry.json`
- `AISE/VERSION` + `AISE/manifest.json` + 最小入口文件

#### AI Agent 项目（参考 agent-workbench）

```text
{project}/
├── src/
│   ├── core/               # Agent 核心运行时
│   ├── skills/             # Skill 实现
│   ├── adapters/           # IDE / API / MCP 适配器
│   └── ui/                 # 可选界面
├── tests/
├── docs/
├── config/
├── mcp/                    # MCP server configs
├── scripts/                # 项目级脚本
├── PROJECT_BLUEPRINT.md
├── CHANGELOG.md
└── .project/
```

#### Trading 项目（参考 jinshi-widget）

```text
{project}/
├── data/                   # 历史数据、缓存
├── strategies/             # 策略实现
├── execution/              # 订单执行
├── risk/                   # 风控
├── dashboard/              # 面板/UI
├── config/
├── tests/
├── PROJECT_BLUEPRINT.md
└── .project/
```

#### Desktop App

```text
{project}/
├── src/
│   ├── ui/                 # 界面层
│   ├── core/               # 业务逻辑
│   ├── platform/           # 平台适配（Windows/macOS/Linux）
│   └── assets/
├── packaging/              # 打包配置
├── tests/
├── docs/
├── PROJECT_BLUEPRINT.md
└── .project/
```

#### Research 项目

```text
{project}/
├── papers/                 # 论文/报告
├── experiments/            # 实验记录
├── notebooks/              # Jupyter notebooks
├── data/                   # 数据集
├── scripts/                # 分析脚本
├── docs/
├── PROJECT_BLUEPRINT.md
└── .project/
```

每种模板应在 `aise-standard/Templates/` 下提供：

- `{type}-PROJECT_BLUEPRINT.template.md`
- `{type}-agent-entry.template.json`
- `{type}-context.template.json`
- `{type}-mission.template.json`
- `{type}-state.template.json`
- `{type}-timeline-event.template.jsonl`
- `{type}-structure.json`（目录树）

`gitops` Skill 创建项目时根据类型选择对应模板。项目最终保留的 `AISE/` 内容遵循 ADR-004：v1.2 Hybrid（manifest + 最小入口），v2.0 Protocol Only。

---

## 9. Backward Compatibility

### 9.1 对 v1.1.2 项目的兼容

- v1.1.2 项目保留完整 `AISE/` 目录，继续工作。
- v1.2 Agent 检测到项目无 `AISE/manifest.json` 时，回退到 v1 加载路径：直接加载完整 `AISE/`。
- `agent_bootstrap.ps1` 同时支持：
  - 老项目：读取 `.agent-entry.json` → 加载完整 AISE。
  - 新项目：读取 `.agent-workspace.json` → 读取 `AISE/manifest.json` → 从 aise-standard 加载。

### 9.2 对 Registry 的兼容

- `repository-identities.json` 保持 schema 稳定，新增字段均为可选。
- `registry/projects.json` 保留为 Cache，v1.2 不再要求手动维护。
- 老 Workspace 无 `.agent-workspace.json` 时，回退到当前行为：从当前目录发现 `.agent-entry.json`。

> 参见 **ADR-002: Project Context Event Sourcing Model**。

### 9.3 对 Handoff 的兼容

- v1.2 起 Canonical Event Source 为 `{project}/.project/context/timeline.jsonl`。
- 支持三种 Handoff 位置：
  - v1.1.2: `agent-governance/.agent-governance/handoff/{project}/latest.json`
  - v1.1.2: `{project}/.project/handoff/HANDOFF.md`
  - v1.2: `{project}/.project/context/handoff/latest.json`（Derived Snapshot）+ `latest.md`（Derived Presentation）
- `agent_exit.ps1` 在 v1.2 中追加事件到 `timeline.jsonl`，并由 AOS Runtime 生成 `.project/context/handoff/` 下的 Snapshot 与 Presentation，同时保留 `.project/handoff/HANDOFF.md` 与 governance index，逐步迁移。

### 9.4 升级路径

- 禁止跨大版本自动升级。
- v1.1.2 → v1.2 通过显式 Migration 脚本执行，需用户确认。
- Migration 前创建 backup tag，失败可回滚。

---

## 10. Protocol Freeze Deliverables（v1.2 三大标准）

本节将 v1.2 范围收缩为三套必须冻结的协议规范，其余实现全部留给 AOS Runtime。

### 10.1 `.agent-entry.json` 标准（v2.0.0）

#### 设计目标

- 任何 Agent、IDE、CLI、Gateway 进入项目时，**第一步只读这一个文件**。
- 文件必须自描述：包含版本、治理指针、运行时声明、必要读取文档。
- 不包含机器相关路径（如 `F:\Agent\...`），机器绑定由 Workspace 文件承担。

#### Schema

```json
{
  "schema_version": "2.0.0",
  "description": "Agent Control Plane Entry — first file every AI agent MUST read",
  "aise": {
    "version": "1.2.0",
    "standard": "lovingxiong-dot/aise-standard",
    "manifest": "AISE/manifest.json"
  },
  "governance": {
    "repo": "lovingxiong-dot/agent-governance",
    "required_reading": [
      ".agent-governance/agent-contract.json",
      ".agent-governance/agent-identity-contract.json",
      ".agent-governance/git-policy.json",
      ".agent-governance/repository-identities.json"
    ]
  },
  "entry_documents": [
    "PROJECT_BLUEPRINT.md",
    "CHANGELOG.md",
    ".project/context.json"
  ],
  "repository": {
    "name": "agent-workbench",
    "github": "lovingxiong-dot/agent-workbench",
    "default_branch": "main",
    "development_branch": "v6-agent",
    "protected_branches": ["main"]
  },
  "branch_strategy": { },
  "rules": { },
  "handoff": {
    "required": true,
    "primary": ".project/context/handoff/latest.json",
    "presentation": ".project/context/handoff/latest.md"
  }
}
```

#### 与 v1.1.2 的差异

| 维度 | v1.1.2 | v1.2 Protocol Freeze |
|------|--------|---------------------|
| 角色 | 指向 agent-governance | 项目级入口 + 治理指针 + AISE 声明 |
| handoff.location | 指向 governance 仓库 | 指向 `.project/context/handoff/` |
| 机器路径 | 可含 `governance_local_path` | **禁止**，由 `.agent-workspace.json` 承担 |
| aise 声明 | 无 | 新增 `aise.version` + `aise.manifest` |

---

> 参见 **ADR-002: Project Context Event Sourcing Model**。

### 10.2 `.project/context` 标准（v1.2.0）

#### 设计目标

- Project Context 不再是 Markdown 文件堆砌，而是 **JSON/Event 驱动的状态层**。
- `timeline.jsonl` 是 Canonical Event Source；`state.json`、`mission.json`、`handoff/latest.json` 均为物化视图。
- Handoff 本质是 **Project State Snapshot**，不是文件。

#### 目录结构

```text
.project/
├── context.json              # 顶层索引
└── context/                  # Project Context 主域
    ├── timeline.jsonl        # Canonical Event Source
    ├── mission.json          # Mission Boundary 物化视图（Derived）
    ├── state.json            # 当前任务状态物化视图（Derived）
    ├── decisions/            # 项目级决策（非 ADR）
    ├── memory/               # 项目记忆
    │   ├── index.json
    │   ├── knowledge/        # 领域知识
    │   ├── architecture/     # 架构文档
    │   ├── patterns/         # 代码模式
    │   └── glossary/         # 术语表
    └── handoff/              # Handoff 物化视图
        ├── latest.json       # Derived Snapshot
        └── latest.md         # Derived Presentation
```

#### `context.json` 示例

```json
{
  "schema_version": "1.2.0",
  "project_id": "agent-workbench",
  "last_updated": "2026-07-15T12:00:00+08:00",
  "pointers": {
    "blueprint": "PROJECT_BLUEPRINT.md",
    "changelog": "CHANGELOG.md",
    "mission": ".project/context/mission.json",
    "state": ".project/context/state.json",
    "timeline": ".project/context/timeline.jsonl",
    "memory_index": ".project/context/memory/index.json",
    "handoff_latest": ".project/context/handoff/latest.json",
    "handoff_presentation": ".project/context/handoff/latest.md"
  },
  "aise": {
    "version": "1.2.0",
    "provider": "agent-gateway"
  }
}
```

#### `state.json` 示例

```json
{
  "schema_version": "1.2.0",
  "project_id": "agent-workbench",
  "current_mission": "Implement Bootstrap Protocol v2",
  "status": "in_progress",
  "blocker": null,
  "completed": [
    "Workspace/Project detection",
    "Git Reality Check order"
  ],
  "remaining": [
    "Handoff JSON schema",
    "Bootstrap v2 integration test"
  ],
  "last_agent": {
    "agent_id": "trae-work-beneficial-wallaby",
    "agent_type": "admin",
    "session_id": "6a50ff3126e74776027a0d15",
    "ended_at": "2026-07-15T11:00:00+08:00"
  },
  "last_commit": "abcd1234",
  "dirty_files": []
}
```

#### Handoff 物化视图

> 参见 **ADR-002: Project Context Event Sourcing Model** 中的 Source of Truth Hierarchy。

- **Canonical Event Source**: `.project/context/timeline.jsonl`
- **Derived Snapshot**: `.project/context/handoff/latest.json`（由 timeline 聚合生成）
- **Derived Presentation**: `.project/context/handoff/latest.md`（由 latest.json 渲染）
- **History**: `.project/context/handoff/history/YYYY/MM/DD-{agent_id}.json`

> 规则：Agent 只追加写入 `timeline.jsonl`；Snapshot 与 Markdown 由 AOS Runtime 或脚本生成。禁止直接编辑 Markdown 或 Snapshot 作为主副本。

---

### 10.3 Bootstrap Protocol v2 标准（1.2.0）

#### 核心变化

| 维度 | v1 | v2 Protocol Freeze |
|------|----|-------------------|
| 入口 | 项目根 `AISE/SYSTEM.md` | `.agent-entry.json` |
| Workspace 识别 | 无 | Step 1 显式检测 |
| 加载顺序 | 先 AISE，后项目 | 先项目现实，后治理规则 |
| Handoff 格式 | Markdown | timeline.jsonl（Canonical）+ latest.json（Derived）+ latest.md（Presentation） |
| Mission Boundary | Step 5 | Step 4，紧接 Git Reality |

#### 14 步协议（冻结版）

```text
Step 1:  Workspace Discovery
    ├── 当前目录存在 .agent-entry.json → Project Bootstrap
    └── 否则 → 读取 .agent-workspace.json → Workspace Bootstrap

Step 2:  Environment Snapshot
    ├── OS / Path / Git availability
    └── 输出 Workspace/Project 选择结果

Step 3:  Load .agent-entry.json
    ├── 解析 aise / governance / repository / handoff 指针
    └── 确定 Governance 仓库本地位置

Step 4:  Git Reality Check
    ├── git status --short
    ├── git branch --show-current
    ├── git rev-parse HEAD
    └── git remote get-url origin

Step 5:  Load Project Context — State
    ├── .project/context/state.json
    └── 识别当前任务 / Blocker / 未完成项

Step 6:  Load Project Context — Handoff
    ├── .project/context/handoff/latest.json
    └── 恢复 Mission / Progress / Next Steps

Step 7:  Load Project Context — Mission Boundary
    ├── .project/context/mission.json
    └── 读取允许/禁止路径与约束

Step 8:  Load Project Context — Memory & Decisions
    ├── .project/context/memory/index.json
    └── .project/context/decisions/

Step 9:  Load Project Goals
    ├── PROJECT_BLUEPRINT.md
    ├── CHANGELOG.md
    └── README.md

Step 10: Resolve Governance
    ├── 通过 .agent-entry.json 定位 agent-governance
    └── 读取 repository-identities.json 交叉验证

Step 11: Load Agent Identity
    ├── agent-governance/.agent-governance/agent-identity-contract.json
    └── 确认当前 Agent 类型、权限、Git 身份

Step 12: Load AISE Runtime
    ├── 读取 AISE/manifest.json
    ├── 通过 .agent-workspace.json 定位 aise-standard
    └── 按需加载 Contracts / Policies / Skills

Step 13: Load Git Governance
    ├── AISE/Git-Governance/git-gate.md
    ├── AISE/Git-Governance/policies.json
    └── 检查 .git/hooks/ 安装状态

Step 14: Compliance Check & Ready
    ├── 执行 AISE Verify
    └── 输出 Bootstrap Report
```

#### Bootstrap Report 模板

```markdown
## AISE Bootstrap Report

- Workspace: <path>
- Project: <project-id>
- AISE Version: 1.2.0
- Governance Repo: lovingxiong-dot/agent-governance
- Agent Mode: Autonomous Agent Engineering
- Git Status: clean / dirty (<N> files)
- Current Branch: <branch>
- Handoff Loaded: yes / no
- Mission Boundary: loaded
- Compliance Status: PASS / WARNING / FAIL
- Status: READY
```

---

## 11. Architecture Decision Records（ADR）

> 本节将本提案中的核心架构决策冻结为 ADR。所有 ADR 进入 **Frozen** 状态后，任何后续实现必须遵守，修改需走架构评审。

### ADR-001: AISE Protocol Layer Separation

**状态**: Accepted — Frozen  
**生效日期**: 2026-07-15  
**决策编号**: ADR-001  
**标题**: AISE 从 Agent 框架降维为 Agent 工程协议层

#### Context

AISE v1.1.x 之前存在向“大型 Agent 框架”演进的趋势：

```text
AISE
 ├── Rules
 ├── Skills
 ├── Templates
 ├── Memory
 └── Agent 行为控制
```

这种结构会导致：
- Claude、Trae、Cursor、Codex、OpenAI Agent 等主流 Agent 不愿被第三方框架接管。
- Registry、Gateway、MCP、Memory 等运行时能力与工程标准混在同一个仓库。
- 标准、运行时、治理、产品四层职责边界模糊，容易互相吞噬。

#### Decision

AISE 从“系统”下降为“协议层”：

```text
AISE Protocol
      |
      ↓
Governance Protocol（治理协议）
      |
      ↓
Project Context
      |
      ↓
Project Runtime
```

v1.2 Protocol Freeze 只规定：

1. `.agent-entry.json` 入口契约
2. `.project/context` 上下文结构
3. Bootstrap Protocol v2 启动顺序
4. Git Governance 规则
5. Engineering Contract

以下能力明确**不属于** AISE：

| 能力 | 负责方 |
|------|--------|
| Project Registry / Identity Service | AOS Runtime / Agent Gateway |
| Capability Router / Policy Engine | AOS Runtime / Agent Gateway |
| MCP Server / Client 运行时 | AOS Runtime / all_mcp_api |
| Memory Service / Memory Index | AOS Runtime / Agent Workbench |
| UI / Session / Dialog | Agent Workbench |
| Remote Agent / Service Discovery | Agent Gateway |

#### Consequences

- ACP / AISE / AOS / Gateway 四层正交，可独立演进。
- AISE 成为可嵌入任何 Agent Runtime 的轻量协议层。
- AOS v7 起作为 **AISE Protocol Consumer** 实现，而不是替代 AISE。
- 新增运行时能力时，先问“这是协议还是实现？”再决定放在哪个仓库。

#### Related

- ADR-003: AOS / Workbench / Gateway Layer Separation
- ADR-004: AISE Distribution Strategy

---

### ADR-002: Project Context Event Sourcing Model

**状态**: Accepted — Frozen  
**生效日期**: 2026-07-15  
**决策编号**: ADR-002  
**标题**: Project Context 采用 Event Sourcing 模型，timeline.jsonl 为 Canonical Event Source

#### Context

Project Context 是 Agent 工程真正的核心资产：

```text
Project Domain:    项目是什么？
Project Context:   项目现在做到哪里？
```

但 v1.1.x 存在以下问题：
- Handoff 以 Markdown 为主副本，机器难以消费。
- `state.json` 等 snapshot 文件一旦丢失或冲突，难以恢复。
- 不同 Agent 之间无法共享细粒度上下文变更历史。

#### Decision

Project Context 采用 **Event Sourcing** 模型：

```text
.project/context/
├── timeline.jsonl          # Canonical Event Source（规范事件源）
├── state.json              # timeline 的物化视图（Derived）
├── mission.json            # Mission Boundary 物化视图（Derived）
└── handoff/
    ├── latest.json         # 由 timeline 聚合生成（Derived）
    └── latest.md           # 展示层（Derived）
```

**Source of Truth Hierarchy**：

```text
Level 0: Git History
              |
              ↓
Level 1: Project Context Event Log (timeline.jsonl)
              |
              ↓
Level 2: Materialized State (state.json / mission.json / handoff/latest.json)
              |
              ↓
Level 3: Presentation (handoff/latest.md)
```

- **Level 0 Git History**：代码、Blueprint、Changelog 的最终真相。
- **Level 1 timeline.jsonl**：Project Context 的 Canonical Event Source，记录 Agent 会话级事件。
- **Level 2 Materialized State**：由 timeline 投影生成的 snapshot，允许删除后重建。
- **Level 3 Presentation**：人类可读展示，禁止作为主副本。

**规则**：

1. Agent 只追加写入 `timeline.jsonl`，不直接修改 `state.json`。
2. `state.json`、`mission.json`、`handoff/latest.json` 由 AOS Runtime 或脚本根据 timeline 重建。
3. `handoff/latest.md` 由 `handoff/latest.json` 渲染，仅用于人类可读展示。
4. 任何 Snapshot 文件允许删除后重建。

事件示例：

```jsonl
{"ts":"2026-07-15T10:00:00+08:00","type":"agent.bootstrap","agent_id":"trae-work-beneficial-wallaby","payload":{"aise_version":"1.2.0"}}
{"ts":"2026-07-15T11:00:00+08:00","type":"task.started","payload":{"mission":"Implement Bootstrap Protocol v2"}}
{"ts":"2026-07-15T12:00:00+08:00","type":"handoff.created","payload":{"status":"in_progress","blocker":null}}
```

#### Consequences

- Project Context 成为可重建、可审计、可共享的持久资产。
- Agent 切换时，新 Agent 可重放事件恢复上下文，而不是依赖上一份 Handoff 的主观摘要。
- Gateway 可消费 `timeline.jsonl` 做跨 Agent 审计与协同。
- 实现复杂度高于纯 snapshot，需要 AOS Runtime 提供 event replay 能力。

#### Related

- Section 3.3: Project Context Model
- Section 10.2: `.project/context` 标准

---

### ADR-003: AOS / Workbench / Gateway Layer Separation

**状态**: Accepted — Frozen  
**生效日期**: 2026-07-15  
**决策编号**: ADR-003  
**标题**: Agent Workbench、AOS Runtime、Agent Gateway 三层解耦

#### Context

AOS / Agent Workbench / Gateway 在讨论中常被混为一谈，导致：
- Workbench 被误认为是 Runtime Owner。
- Gateway 被误认为是 AOS 本身。
- Runtime 能力不知道该放在 Workbench、AOS 还是 Gateway。

#### Decision

明确拆分为三层：

```text
Agent Workbench
      |
      | 人机交互 + 项目管理 + 可视化验证
      ↓
AOS Runtime
      |
      | Agent 运行环境、Session、Capability、Skill Execution
      ↓
Agent Gateway
      |
      | MCP、External API、Model Router、Remote Agent、Service Discovery
```

#### 各层职责

| 层级 | 职责 | 示例 | 禁止 |
|------|------|------|------|
| **Agent Workbench** | AISE 官方产品化验证平台 | UI、Workspace、Inspector、Navigator、Chat | 直接调用 Capability / Provider / Service；拥有 Runtime |
| **AOS Runtime** | Agent 运行环境 | SessionManager、AgentRuntime、CapabilityRuntime、SkillRuntime | 直接暴露 MCP Server；处理远程服务发现 |
| **Agent Gateway** | 分布式服务层 | MCP Gateway、Model Router、External API、Remote Agent、Identity Service | 拥有 UI；执行 Skill；管理 Project Context |

#### 数据流

```text
User / UI / MCP / Remote Agent
        |
        ↓
Agent Workbench  ── 消费 AISE Protocol
        |
        ↓
AOS Runtime      ── 执行 Capability / Skill / Workflow
        |
        ↓
Agent Gateway    ── 路由到 External API / Model / Remote Agent
```

#### Consequences

- Workbench 从 Runtime Owner 改造为 Runtime Client。
- AOS Runtime 不依赖具体 UI 实现，可被 Workbench / CLI / Web / Cloud 多种前端消费。
- Gateway 作为独立服务层，未来可部署为远程服务。
- 新增能力时，先确定属于哪一层再实现。

#### Related

- ADR-001: AISE Protocol Layer Separation
- ADR-004: AISE Distribution Strategy

---

### ADR-004: AISE Distribution Strategy

**状态**: Accepted — Transitional  
**生效日期**: 2026-07-15  
**决策编号**: ADR-004  
**标题**: AISE 分发策略 — v1.2 Hybrid Runtime，v2.0 Protocol Only

#### Context

AISE 标准如何进入项目存在多种方案：

- Option A: 每个项目复制完整 `AISE/`
- Option B: 中央 `aise-standard` + 项目轻量 Runtime
- Option C: 混合模式（manifest + 中央源 + 最小入口文件）
- Option D: 纯协议指针（项目不保留 `AISE/`）

需要在 v1.2 阶段选择一种可冻结、可演进、可落地的策略。

#### Decision

采用**两阶段策略**：

**v1.2 Protocol Freeze：Hybrid（混合模式）**

项目保留：

```text
AISE/
├── VERSION
├── manifest.json
└── Agent-Entry/        # AGENTS.md / CLAUDE.md / .trae/rules/aise.md
```

中央 `aise-standard` 保留完整协议内容，作为唯一 Source of Truth。

**v2.0：Protocol Only（纯协议指针）**

项目不再保留任何 `AISE/` 目录：

```text
project/
├── .agent-entry.json
├── .project/
├── .agent/
└── source
```

`.agent-entry.json` 声明：

```json
{
  "schema_version": "2.0.0",
  "aise_version": "1.2.0",
  "provider": "agent-gateway",
  "runtime": "aise"
}
```

类似：

```text
package.json  → npm
Dockerfile    → Docker
.agent-entry.json → AISE
```

#### Consequences

- v1.2 保留自发现能力和离线回退，降低迁移风险。
- v2.0 项目零 AISE 代码，标准升级完全外部化。
- AOS Runtime / Agent Gateway 负责解析 `.agent-entry.json` 并加载对应协议版本。
- 禁止 v1.2 直接跳到 v2.0，必须通过显式 Migration。

#### Related

- Section 7: AISE Protocol Distribution Model
- ADR-001: AISE Protocol Layer Separation
- ADR-003: AOS / Workbench / Gateway Layer Separation

---

## 附录：核心设计决策总结

| 决策 | 推荐方案 | ADR | 原因 |
|------|---------|-----|------|
| AISE 范围 | 从框架降维为协议层 | ADR-001 | 避免 AISE 成为 Vendor Lock-in 的全家桶框架 |
| Project Context | Event Sourcing：timeline.jsonl 为 Canonical Event Source | ADR-002 | 状态可重建、可审计、可跨 Agent 共享 |
| Workbench/AOS/Gateway | 三层解耦 | ADR-003 | UI / Runtime / 分布式服务可独立演进 |
| AISE 分发 | v1.2 Hybrid → v2.0 Protocol Only | ADR-004 | 项目最终零 AISE 代码，标准外部化 |
| Workspace 与 Project 区分 | 引入 `.agent-workspace.json` | - | 解决进入 Workspace 根目录 Bootstrap 失败的问题 |
| Registry 重复 | 仅保留 `repository-identities.json` 为 Primary；其余为 Cache | - | 消除双写，支持多 Workspace |
| Handoff 来源 | 项目本地 `.project/context/timeline.jsonl`（Canonical）+ `handoff/latest.json`（Derived）+ `latest.md`（Presentation） | ADR-002 | JSON 事件源为 Primary，Markdown 仅展示 |
| Bootstrap 顺序 | Git Reality → Handoff → Project Goals → AISE Runtime | - | Mission 驱动，先理解项目再加载规则 |
| Project Context | 统一 `.project/` 结构，新增 `context.json` | ADR-002 | 消除 `aise-init.ps1` 与 Policy 的不一致 |
| 入口文件重复 | 保留 IDE 入口文件，内容由中央模板统一生成 | - | 保持 IDE 自发现，减少手动同步 |
| AISE v1.2 范围 | Protocol Freeze：只冻结 3 套协议 | ADR-001~004 | 防止 AISE / AOS / Gateway 职责互相吞噬 |

---

**数据来源**：

- `aise-standard/SYSTEM.md`
- `aise-standard/Agent-Entry/Bootstrap.md`
- `aise-standard/Policies/memory-ownership.md`
- `aise-standard/Policies/mission-boundary.md`
- `agent-governance/PROJECT_BLUEPRINT.md`
- `agent-governance/README.md`
- `agent-governance/.agent-governance/registry/projects.json`
- `agent-governance/.agent-governance/repository-identities.json`
- `agent-governance/.agent-governance/workspace-registry.json`
- `agent-governance/.agent-governance/agent-contract.json`
- `agent-governance/.agent-governance/agent-identity-contract.json`
- `agent-governance/scripts/aise-init.ps1`
- `agent-governance/scripts/agent_bootstrap.ps1`
- `agent-governance/aise-template/`
- `gateway/all_mcp_api/PROJECT_BLUEPRINT.md`
- `gateway/all_mcp_api/docs/gateway_framework.md`
- `workbench/agent_workbench/PROJECT_BLUEPRINT.md`
- `workbench/agent_workbench/.agent-entry.json`
- `workbench/agent_workbench/.project/decisions/ai-software-engineering-workflow.md`
- `workbench/agent_workbench/.project/contracts/repository_contract.md`
- `workbench/agent_workbench/docs/engineering-workflow.md`
- `workbench/agent_workbench/docs/v6/SPEC.md`
- `workbench/agent_workbench/docs/v6/ROADMAP.md`
