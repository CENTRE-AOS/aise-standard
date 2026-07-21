# CENTRE Runtime Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: CENTRE Gateway Runtime Control Plane v3.2.0
> 前置: CENTRE Gateway Runtime Constitution v1.0.0frozen

---

## 1. 定义

CENTRE Runtime Contract 定义 Runtime 内核必须对外暴露的稳定接口边界。任何实现 CENTRE Runtime 的组件必须遵守此 Contract。

Runtime 内核是 CENTRE Gateway Runtime 的不可变核心，负责：

- 状态机调度
- Skill 生命周期管理
- 事件路由
- 审计追踪

## 2. 接口边界

### 2.1 CLI Interface

Runtime 必须通过统一 CLI 入口暴露管理命令：

```
centre <command> [options]
```

CLI 是薄包装器。规则定义在 Protocol 中，CLI 只负责分发。

**管理命令（对外暴露）：**

| 命令 | 职责 |
|------|------|
| `bootstrap` | 初始化 Runtime 环境 |
| `install` | 安装 Protocol Artifact |
| `check` | 健康检查（环境 + 协议 + Runtime + Skill） |
| `status` | 环境状态 |
| `version` | 版本信息 |
| `skills` | Skill 管理（list/status/verify/detail/activate/deactivate/install） |
| `trigger` | State Machine 事件触发 |
| `handoff` | Agent 交接（上下文导出） |

**约束：**

- CLI 不得暴露 `skill run` 或直接 Skill 执行入口
- 所有 Skill 执行必须通过 `trigger` → StateMachine → SkillManager 路径
- CLI 版本号独立于 Runtime 版本号

### 2.2 State Interface

Runtime 必须提供 State Machine 引擎：

```
Event → StateMachine → SkillManager → Skill Execution → Audit → State Update
```

**已知状态（13 个）：**

```
agent.enter, agent.exit, project.created, project.attach,
handoff.request, release.triggered, pre-commit, pre-push,
protocol.check, environment.init, environment.sync,
runtime.health, runtime.update
```

**禁止的触发状态（5 个）：**

```
admission.passed, admission.denied, handoff.completed,
action.completed, action.failed
```

这些状态由 Skill 执行产生，不可作为外部触发源。

**事件链：**

```
agent.enter       → admission.passed
agent.exit        → handoff.completed
release.triggered → action.completed
```

### 2.3 Skill Interface

Runtime 必须通过 SkillManager 提供统一的 Skill 管理：

```
SkillManager
├── Initialize-SkillCache   (内部)
├── Get-SkillStatus         (管理)
├── Get-SkillDetail         (管理)
├── Invoke-VerifyAllSkills  (管理)
├── Set-SkillState          (管理)
├── Resolve-SkillsByState   (内部，StateMachine 调用)
├── Invoke-SkillsByState    (内部，StateMachine 调用)
├── Invoke-SkillDirect      (内部)
└── Update-SkillCache       (管理)
```

### 2.4 Registry Interface

Runtime 必须维护以下 Registry：

| Registry | 路径 | 职责 |
|----------|------|------|
| Skill Registry | `system-skills/registry.json` | 已安装 Skill 索引 |
| Adapter Registry | `registry/adapters.json` | Agent 适配器映射 |
| Protocol Registry | `registry/protocol_versions.json` | 协议版本兼容性 |
| Project Registry | `registry/installed_projects.json` | 已注册项目 |

### 2.5 Audit Interface

Runtime 必须记录所有状态转换事件：

```
.project/audit/gate-events.jsonl
```

每条事件包含：

```json
{
  "timestamp": "ISO8601",
  "event": "state.event_name",
  "source": "component",
  "result": "success|failure",
  "context": {}
}
```

### 2.6 Bootstrap Interface

Runtime 必须在启动时加载 Bootstrap Contract：

```
Agent Arrival
      │
      ▼
AGENTS.md           ← Layer 1: Universal Agent Instruction (行为规则)
      │
      ▼
AGENT_CONTEXT.md    ← Layer 2: CENTRE Identity Contract (世界模型)
      │
      ▼
PROJECT_STATE.json  ← Layer 3: Repository State (当前状态)
      │
      ▼
PROJECT_BLUEPRINT.md ← Layer 4: Architecture Truth (架构真相)
```

**Bootstrap 失败条件**：
- AGENTS.md 不存在 → 降级为只读模式，Agent 不可修改任何文件
- AGENT_CONTEXT.md 不存在 → 终止，Agent 不知道自己在哪
- Authority Ranking 无法建立 → 终止，Agent 无法区分真实与历史
- Repository Identity 不明确 → 降级为只读模式

**必须加载的文件**：
- `AGENTS.md` — Universal Agent Instruction
- `AGENT_CONTEXT.md` — CENTRE Identity Contract

**Bootstrap 生命周期**（v3.2.0 新增）：

Bootstrap Engine 支持三种模式：

```
Bootstrap Engine
       │
       ├── BIRTH    — Agent 首次进入工作区
       ├── RESTORE  — Agent 接替 Handoff 恢复上下文
       └── RECOVERY — Agent 上下文损坏后重建
```

| 模式 | 触发条件 | 输入 | 输出 |
|------|---------|------|------|
| BIRTH | Agent 首次进入 | 工作区文件系统 | Bootstrap Report |
| RESTORE | Handoff 接替 | Handoff Bootstrap State + 当前文件系统 | Restore Report (RESTORED/PARTIAL/FAILED) |
| RECOVERY | 上下文损坏 | 最小可用资产 | Recovery Report (PARTIAL_RECOVERY/MINIMAL_RECOVERY/FAILED) |

**RESTORE 流程**：
1. 读取 Handoff Bootstrap State（移交时的 AGENTS.md/AGENT_CONTEXT.md 完整性、Authority Level、Source Authority）
2. 对比当前 AGENTS.md/AGENT_CONTEXT.md 是否与移交时一致
3. 验证上下文连续性
4. 重新建立 Context Boundary

**约束**：
- 只有一个 Bootstrap Engine 实现（aise-bootstrap），不允许 handoff 或其他 Skill 自行实现 Bootstrap Restore
- Handoff RESUME 必须调用 `aise-bootstrap.restore()` 而非自己验证 Bootstrap Context
- Bootstrap Restore 是 Admission 的前置条件

### 2.7 Governance Loop Interface

Runtime 必须实现六阶段治理闭环：

```
OBSERVE → EVALUATE → DECIDE → EXECUTE → RECORD → UPDATE
```

定义详见 `runtime/kernel/governance-loop/governance-loop.md`（RFC-0008）。

Governance Loop 在 Bootstrap Phase 完成后启动，确保 Agent 在正确的上下文和 Authority 下执行操作。

## 3. 版本模型

CENTRE 采用三层版本模型：

| 层 | 版本 | 生命周期 | 职责 |
|----|------|---------|------|
| Protocol | 2.0.0-frozen | 慢 | Constitution, Contract, Identity Schema, Communication Rules |
| Runtime | 3.2.0 | 中 | StateMachine, SkillManager, Registry, Event Engine, Governance Loop, Bootstrap |
| CLI | 3.2.0 | 快 | install, verify, skills, handoff |

版本声明位置：

- Protocol: `aise-standard` (GitHub Release Artifact)
- Runtime: `VERSION`, `.agent-entry.json`, `protocol_versions.json`
- CLI: `runtime/cli/centre.ps1`

## 4. 不可违背约束

1. Runtime 不定义协议 — 协议定义属于 aise-standard
2. Runtime 不拥有项目资产 — 资产属于 Project
3. Runtime 不直接暴露 Skill 执行 — 必须通过 StateMachine
4. Skill 归属 Runtime，不属于 Agent
5. 所有状态转换必须通过 Audit 记录
6. CLI 是薄包装器，规则在 Protocol 中
7. Runtime 启动前必须完成 Bootstrap Phase（加载 AGENTS.md + AGENT_CONTEXT.md）