# CENTRE Event Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: CENTRE Gateway Runtime Control Plane v0.1
> 前置: Runtime Contract v1.0.0

---

## 1. 定义

CENTRE Event Contract 定义 Runtime 内部事件的标准格式、路由规则和生命周期。Event 是 StateMachine 的唯一输入，是 Skill 的唯一触发源。

## 2. Event Schema

### 2.1 标准 Event 结构

```json
{
  "id": "evt-<uuid>",
  "source": "component",
  "type": "state.event_name",
  "timestamp": "ISO8601",
  "payload": {},
  "context": {
    "project_id": "string",
    "agent_id": "string",
    "tenant_id": "string"
  },
  "trace": {
    "parent_id": "string | null",
    "chain": ["string"]
  }
}
```

### 2.2 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 全局唯一事件 ID |
| `source` | string | 是 | 事件来源组件（cli, adapter, skill, system） |
| `type` | string | 是 | 事件类型，对应已知状态 |
| `timestamp` | string | 是 | ISO8601 格式时间戳 |
| `payload` | object | 否 | 事件携带数据 |
| `context` | object | 否 | 事件上下文（project, agent, tenant） |
| `trace` | object | 否 | 分布式追踪信息 |

## 3. Event 类型

### 3.1 外部触发事件（13 个）

由 CLI 或 Adapter 触发，进入 StateMachine：

```
agent.enter            Agent 进入项目
agent.exit             Agent 退出项目
project.created        项目创建
project.attach         项目挂载
handoff.request        交接请求
release.triggered      发布触发
pre-commit             Git pre-commit
pre-push               Git pre-push
protocol.check         协议检查
environment.init       环境初始化
environment.sync       环境同步
runtime.health         Runtime 健康检查
runtime.update         Runtime 更新
```

### 3.2 内部事件（5 个）

由 Skill 执行产生，不可作为外部触发源：

```
admission.passed       准入通过
admission.denied       准入拒绝
handoff.completed      交接完成
action.completed       操作完成
action.failed          操作失败
```

### 3.3 降级事件

Skill 执行异常时发射：

```
skill.degraded         Skill 降级运行
skill.error            Skill 执行错误
runtime.panic          Runtime 异常
```

## 4. Event 路由

```
Source (CLI/Adapter)
    │
    ▼
StateMachine.Test-State()
    │
    ├── 禁止状态 → 拒绝
    │
    ▼
StateMachine.Invoke-StateEvent()
    │
    ▼
SkillManager.Invoke-SkillsByState()
    │
    ├── Skill 1 → Skill 2 → ... → Skill N
    │
    ▼
Audit (gate-events.jsonl)
    │
    ▼
EventChain (后续状态)
    │
    ├── admission.passed
    ├── handoff.completed
    └── action.completed
```

## 5. Event Chain

事件链由 StateMachine 驱动，不由 Skill 自行触发：

```
agent.enter       → admission.passed  (admission 通过后触发 inject)
agent.exit        → handoff.completed (handoff 完成后标记)
release.triggered → action.completed
```

后续状态不触发 Skill（它们是内部状态标记）。

## 6. Audit

所有事件必须记录到：

```
.project/audit/gate-events.jsonl
```

格式：

```jsonl
{"timestamp":"2026-07-17T16:00:00+08:00","event":"agent.enter","source":"cli","result":"success","context":{"project_id":"aos-runtime"}}
```

## 7. 不可违背约束

1. Event 是 StateMachine 的唯一输入
2. 禁止自我触发循环（admission 不监听 admission.passed）
3. 事件链由 StateMachine 驱动，不由 Skill 自行触发
4. 所有事件必须经过 Audit 记录
5. 内部事件不可作为外部触发源
6. 降级事件必须记录但不阻塞流程