# CENTRE Artifact Consumption Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-21
> 适用范围: CENTRE Foundation v3.2.0
> 前置: Artifact Contract v1.0.0, Installer Contract v1.0.0, Runtime Contract v1.0.0
> 关联: runtime-consumption-authority-map.json, artifact-contract.md

---

## 1. 定义

CENTRE Artifact Consumption Contract 定义 Runtime Instance 如何消费已安装的 Artifact。Runtime 是 Artifact 的消费者，不是 Artifact 的修改者、生产者或分发者。

**Runtime 是 Artifact Consumer，不是 Artifact Authority。**

```
A4 Install Authority
        │
        │ creates
        ▼
Installed Artifact [artifact_state: installed]
        │
        │ read-only mount
        ▼
Runtime Instance
        │
        │ consumes
        ▼
Runtime Execution
```

## 2. Runtime Consumption Identity

| 属性 | 值 |
|------|-----|
| Role | Artifact Consumer |
| Owner | aos-runtime |
| 职责 | 读取已安装 Artifact、加载 Runtime 组件、验证兼容性、执行 Agent 生命周期 |
| 输入 | Installed Artifact (artifact_state = installed) |
| 产出 | Runtime Execution (active instance) |
| 禁止 | 修改 Artifact、重建 Artifact、签名 Artifact、绕过 Installer 加载组件 |

### 2.1 Runtime Consumption 不是什么

| 不是 | 原因 |
|------|------|
| 不是 Artifact Producer | Build 属于 A2 (aos-factory-new) |
| 不是 Artifact Signer | Sign 属于 A3 (Artifact Authority) |
| 不是 Artifact Installer | Install 属于 A4 (aos-installer) |
| 不是 Source Consumer | Runtime 不直接消费 Source Repository |
| 不是 Protocol Authority | Protocol 属于 aise-standard (A0) |

## 3. Runtime Lifecycle（独立于 Artifact Lifecycle）

Artifact Lifecycle 和 Runtime Lifecycle 是两种不同的状态模型，描述不同对象的不同维度。

### 3.1 Runtime Lifecycle States

```
created
    │
    │ Initialize
    ▼
initialized
    │
    │ Activate
    ▼
active
    │
    │ Suspend
    ▼
suspended
    │
    │ Resume / Terminate
    ▼
terminated
```

| State | 含义 | 触发者 | 下一状态 |
|-------|------|--------|---------|
| `created` | Runtime Instance 已由 A4 Installer 创建，尚未初始化 | A4 Installer | `initialized` |
| `initialized` | Bootstrap 完成，组件已加载，尚未激活 | Runtime | `active` |
| `active` | Runtime 正在执行，Agent 可以交互 | Runtime | `suspended` / `terminated` |
| `suspended` | Runtime 暂停，保持状态，等待恢复 | Runtime | `active` / `terminated` |
| `terminated` | Runtime 已停止，可重建 | Runtime | `created` (重建) |

### 3.2 与 Artifact Lifecycle 的边界

```
Artifact Lifecycle (artifact_state):
  candidate → signed → verified → installed (终态)

Runtime Lifecycle (runtime_state):
  created → initialized → active → suspended → terminated
```

**交汇点**：
- `installed` (Artifact) = `created` (Runtime) 的起点
- Artifact 在 `installed` 后不再改变状态
- Runtime 在 `created` 后开始独立的生命周期

**禁止混合**：

```
FORBIDDEN:
  artifact_state = "active"          (active 是 Runtime 状态)
  runtime_state = "installed"        (installed 是 Artifact 状态)
  Runtime State 影响 Artifact State
  Artifact State 影响 Runtime State
```

## 4. Runtime Input Contract

### 4.1 唯一合法输入

```
Runtime 唯一输入:

Installed Artifact
  - 来源: A4 Install Authority
  - 位置: AOS_HOME (由 Installer 创建)
  - 必须满足: artifact_state = installed
  - 必须包含: artifact.manifest.json
```

### 4.2 禁止的输入

```
FORBIDDEN:

  Source Repository (git clone)
  Git URL
  Artifact Candidate (artifact_state = candidate)
  Unsigned Artifact (artifact_state = signed)
  Unverified Artifact (artifact_state = verified)
  Build Output Directory
  Package Registry Source Code
  Foreign File System paths
```

## 5. Runtime Boot Contract

### 5.1 启动流程

```
Runtime Start
    │
    ▼
Step 1: Locate Installed Artifact
  读取 AOS_HOME 中的 artifact.manifest.json
    │
    ▼
Step 2: Read Manifest
  读取 artifact_id, artifact_version, artifact_type
  读取 version_model (protocol/runtime/cli/artifact versions)
  读取 compatibility
  读取 bootstrap_contract
    │
    ▼
Step 3: Validate Compatibility
  验证 runtime_version 与当前期望一致
  验证 protocol_version 兼容
  验证 bootstrap_contract 满足 (AGENTS.md + AGENT_CONTEXT.md)
    │
    ▼
Step 4: Create Runtime Context
  初始化 Runtime State = created
  加载 Kernel 组件
    │
    ▼
Step 5: Load Components
  加载 Skills (从 Artifact 中的 Skills/ 目录)
  加载 System Skills (从 Artifact 中的 system-skills/ 目录)
  加载 Adapters
    │
    ▼
Step 6: Activate
  Runtime State: created → initialized → active
  开始接收 Agent Intent
```

### 5.2 启动约束

- 所有组件必须从 Installed Artifact 加载，不通过 git clone/npm install/pip install
- 启动失败 → 回滚到 initialized 状态，记录错误
- 不假设 Artifact 之外的任何文件存在

## 6. Runtime Consumption Model

### 6.1 允许的操作

```
ALLOWED:

  READ artifact.manifest.json
  READ artifact content (runtime/*, Skills/*, system-skills/*)
  LOAD runtime modules
  LOAD Skills
  LOAD System Skills
  VERIFY runtime compatibility
  VERIFY bootstrap_contract
  EXECUTE Agent Lifecycle
  EXECUTE Skill Lifecycle
  REPORT Runtime State
```

### 6.2 禁止的操作

```
FORBIDDEN:

  # Artifact Modification
  MODIFY artifact
  DELETE artifact
  REPLACE artifact
  REBUILD artifact
  PATCH artifact
  CHANGE artifact_state

  # Source Consumption
  GIT CLONE source repository
  GIT PULL source repository
  READ source repository as runtime dependency
  READ git tag for version
  READ git commit for state

  # Authority Violation
  SIGN artifact
  CHANGE manifest authority
  CHANGE version_model
  CHANGE compatibility declaration

  # Self-Evolution
  DOWNLOAD update
  REPLACE itself
  PATCH kernel
  REBUILD from source
```

### 6.3 Version Consumption

```
正确:
  version = artifact.manifest.json.version_model.runtime_version

错误:
  version = git describe --tags
  version = VERSION file (standalone)
  version = package.json version
  version = runtime.manifest.json version
```

Runtime 版本号必须从 `artifact.manifest.json` 的 `version_model.runtime_version` 读取，不通过 git tag、VERSION 文件或其他独立来源。

## 7. Skill Loading Contract

### 7.1 Skill 加载路径

```
Installed Artifact
    │
    │ artifact.manifest.json
    │
    ├── Skills/
    │   ├── aise-bootstrap/
    │   ├── aise-admission/
    │   └── ...
    │
    └── system-skills/
        ├── 10-lifecycle/
        ├── 20-governance/
        └── ...
```

```
正确:
  Skill ← Artifact/Skills/ ← artifact.manifest.json

错误:
  Skill ← Git Repository ← git clone
  Skill ← File System ← arbitrary path
  Skill ← npm registry ← npm install
```

### 7.2 Skill 加载约束

- Skill 必须从 Installed Artifact 的 Skills/ 目录加载
- 不通过 git clone、npm install、pip install 加载 Skill
- Skill 的 SKILL.md 必须定义 Authority Level（遵循 skill-authority-map.json）
- Skill 执行前必须通过 Bootstrap Context Check

## 8. Upgrade Boundary

### 8.1 正确升级路径

```
New Artifact (A3: Signed)
    │
    ▼
A4 Installer
    │
    │ 7-stage Pipeline
    ▼
New Runtime Instance
    │
    │ Activate
    ▼
Active Runtime
    │
    │ Decommission
    ▼
Old Runtime Instance → terminated
```

### 8.2 禁止的升级路径

```
FORBIDDEN:

  Runtime download update
  Runtime replace itself
  Runtime patch kernel
  Runtime rebuild from source
  Runtime git pull && restart
  Runtime npm update && restart
```

### 8.3 Self-Modification Guard

Runtime 自修改（升级/替换）必须经过完整的 Authority Check 链：

```
1. Proposal → Agent 提交 artifact.install intent
2. Governance Decision → REQUIRE_HUMAN (constitutional)
3. Artifact Verification → SHA256 + Signature + Compatibility
4. Human Approval → REQUIRED
5. A4 Installer → 7-stage Pipeline → New Runtime Instance
6. Decommission → Old Runtime → terminated
```

Runtime 不可自我升级。升级必须通过 A4 Installer 创建新 Runtime Instance。

## 9. 与现有 Contract 的关系

### 9.1 与 Artifact Contract 的关系

- Artifact Contract §3.5 定义 Artifact Lifecycle vs Runtime Lifecycle 边界
- Consumption Contract 在此基础上定义 Runtime Lifecycle 的具体状态
- artifact_state.installed 是 Artifact 的终态，也是 Runtime 的起点

### 9.2 与 Installer Contract 的关系

- Installer Contract §4 定义 Installed Artifact 的产出
- Consumption Contract 定义 Runtime 如何消费该产出
- A4 Installer 和 Runtime 通过 Installed Artifact 交接，无直接 API 调用

### 9.3 与 Runtime Contract 的关系

- Runtime Contract §2 定义 Runtime 的 CLI Interface 和 Registry Interface
- Consumption Contract 定义 Runtime 如何从 Artifact 加载这些组件
- Runtime Contract §3 版本模型 — Consumption Contract 强制从 Manifest 读取版本

### 9.4 与 Governance Loop 的关系

- update-loop.md §5.6 Self-Modification Guard 已定义 Runtime 自修改的 Authority Check 链
- Consumption Contract 在此基础上增加 Upgrade Boundary 约束

## 10. 不可违背约束

1. Runtime 不消费 Source — 只消费 Installed Artifact
2. Runtime 不修改 Artifact — 只读访问
3. Runtime 不签名 Artifact — 签名属于 A3
4. Runtime 不自我升级 — 升级必须通过 A4 Installer 创建新 Instance
5. Runtime 版本从 Manifest 读取 — 不从 git tag/VERSION 文件读取
6. Skill 从 Artifact 加载 — 不通过 git clone/npm install
7. Runtime Lifecycle 独立于 Artifact Lifecycle — 两种状态模型，不混合
8. Runtime 启动必须验证 Bootstrap Contract — AGENTS.md + AGENT_CONTEXT.md
9. Runtime 不假设 Artifact 之外的任何文件存在
10. Runtime 不自我判断 Identity/Permission/Policy — 委托 CENTRE Kernel