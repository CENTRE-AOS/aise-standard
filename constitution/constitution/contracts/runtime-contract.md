# CENTRE Runtime Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: CENTRE Gateway Runtime Control Plane v0.1
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

## 3. 版本模型

CENTRE 采用三层版本模型：

| 层 | 版本 | 生命周期 | 职责 |
|----|------|---------|------|
| Protocol | 2.0.0-frozen | 慢 | Constitution, Contract, Identity Schema, Communication Rules |
| Runtime | 2.1.0 | 中 | StateMachine, SkillManager, Registry, Event Engine |
| CLI | 2.2.0 | 快 | install, verify, skills, handoff |

版本声明位置：

- Protocol: `aos-protocol-factory` (GitHub Release Artifact)
- Runtime: `VERSION`, `.agent-entry.json`, `protocol_versions.json`
- CLI: `runtime/cli/centre.ps1`

## 4. 不可违背约束

1. Runtime 不定义协议 — 协议定义属于 aos-protocol-factory
2. Runtime 不拥有项目资产 — 资产属于 Project
3. Runtime 不直接暴露 Skill 执行 — 必须通过 StateMachine
4. Skill 归属 Runtime，不属于 Agent
5. 所有状态转换必须通过 Audit 记录
6. CLI 是薄包装器，规则在 Protocol 中