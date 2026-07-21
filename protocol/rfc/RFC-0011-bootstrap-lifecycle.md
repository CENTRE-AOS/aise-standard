# RFC-0011: Bootstrap Lifecycle Model

> Status: FROZEN
> Type: Architecture
> Layer: Protocol / Runtime
> Protocol: CENTRE Protocol 2.0.0-frozen
> Foundation: v3.2.0
> Date: 2026-07-20
> Supersedes: None
> Related: RFC-0008 (Governance Loop), RFC-0010 (Context Authority Model)

---

## 摘要

定义 Bootstrap Engine 的三态生命周期模型（BIRTH / RESTORE / RECOVERY），作为 Runtime Identity Continuation Protocol 的核心实现。

---

## 背景

CENTRE Runtime 的 Bootstrap 最初设计为 Agent 首次进入时的单次检查。随着 Handoff 和 Multi-Agent 场景的出现，Bootstrap 需要支持三种入口路径：

1. **BIRTH**: Agent 首次进入工作区，从零建立 Runtime Identity
2. **RESTORE**: Agent 接替 Handoff，从 Snapshot 恢复 Runtime Identity
3. **RECOVERY**: Agent 上下文损坏，从最小可用资产重建 Runtime Identity

同一个 Bootstrap Engine 必须统一管理这三种路径，避免 Handoff 或其他 Skill 自行实现 Bootstrap 逻辑导致身份分裂。

---

## 设计

### Bootstrap Engine 三态模型

```
Bootstrap Engine
       │
       ├── BIRTH    — Agent 首次进入工作区
       ├── RESTORE  — Agent 接替 Handoff 恢复上下文
       └── RECOVERY — Agent 上下文损坏后重建
```

### 模式 BIRTH

**用途**: Agent 首次进入 Runtime。

**触发条件**:
- Agent 进入新工作区
- 无 Handoff 上下文
- 检测到 AGENTS.md / AGENT_CONTEXT.md / .agent-entry.json

**输入**:
- 工作区文件系统
- AGENTS.md（Layer 1: Universal Agent Instruction）
- AGENT_CONTEXT.md（Layer 2: CENTRE Identity Contract）
- PROJECT_STATE.json（Layer 3: Repository State）
- PROJECT_BLUEPRINT.md（Layer 4: Architecture Truth）

**流程**:
1. Authority Ranking（6 级文件优先级）
2. 双层 Bootstrap（AGENTS.md → AGENT_CONTEXT.md）
3. Context Boundary 建立
4. Repository Identity 解析
5. CENTRE 治理项目检测
6. 完整 CENTRE Bootstrap（七步加载链）或最小安全基线（C1-C9）

**输出**: Bootstrap Report（Authority Level, Context Boundary, Repository Identity）

**失败条件**:
- AGENTS.md 缺失 → 降级为只读模式
- AGENT_CONTEXT.md 缺失 → 终止
- Authority Ranking 无法建立 → 终止

### 模式 RESTORE

**用途**: Agent 接替 Handoff 后恢复 Runtime Identity。

**触发条件**:
- Handoff RESUME 调用 `aise-bootstrap.restore(handoff_path)`
- Handoff 包含有效的 Bootstrap State Snapshot

**输入**:
- Handoff Bootstrap State（移交时的 AGENTS.md/AGENT_CONTEXT.md 完整性、Authority Level、Source Authority）
- 当前 AGENTS.md
- 当前 AGENT_CONTEXT.md
- 当前 PROJECT_STATE.json

**流程**:
1. 读取 Handoff Bootstrap State（R1）
2. 对比当前状态（R2）
3. 验证上下文连续性（R3）
4. 重新建立 Context Boundary（R4）
5. 输出 Restore Report（R5）

**输出**: Restore Report（RESTORED / PARTIAL / FAILED）

**恢复结果**:

| 对比结果 | 状态 | 操作 |
|---------|:----:|------|
| Bootstrap State 完全一致 | RESTORED | 进入 Admission → 继续执行 |
| AGENTS.md 不变，其他文件有差异 | PARTIAL | 标记 WARNING，进入 Admission |
| AGENTS.md 变更 | REVIEW | Agent 需确认是否接受新规则 |
| AGENT_CONTEXT.md 变更 | REVIEW | 重新确认 Authority Level |
| 关键文件缺失 | FAILED | 降级为 RECOVERY 模式 |
| Repository Identity 不匹配 | FAILED | 终止，Agent 在错误仓库 |

### 模式 RECOVERY

**用途**: Bootstrap Context 损坏或冲突时，从最小可用信息重建上下文。

**触发条件**:
- RESTORE 失败
- Bootstrap 文件损坏
- 上下文冲突

**流程**:
1. 检测可用资产（C1）
2. 从最小可用资产重建（C2）
3. 标记 RECOVERY 状态（C3）
4. 输出 Recovery Report（C4）

**恢复结果**:

| 可用资产 | 状态 | 操作 |
|---------|:----:|------|
| AGENTS.md + AGENT_CONTEXT.md | PARTIAL_RECOVERY | 只读模式，建议补全 |
| 仅 AGENTS.md | MINIMAL_RECOVERY | 只读模式，不可执行写操作 |
| 仅 AGENT_CONTEXT.md | DEGRADED_RECOVERY | 终止，无行为规则 |
| 都不存在 | FAILED | 终止 |

---

## 约束

### 单一责任

Bootstrap Engine 实现必须唯一（aise-bootstrap）。不允许以下组件自行实现 Bootstrap 逻辑：

- handoff — 必须调用 `aise-bootstrap.restore()` 而非自己验证
- admission — 消费 Bootstrap State，不自行建立
- 任何其他 Skill — 不实现 Bootstrap 检查

### 生命周期顺序

```
BIRTH/RESTORE/RECOVERY
        │
        ▼
    Admission
        │
        ▼
    Execution
        │
        ▼
    Handoff SAVE
        │
        ▼
    RESTORE (next Agent)
```

Bootstrap 始终是 Admission 的前置条件。Bootstrap 失败不阻断后续工作，但必须限制 Agent 权限。

### 版本

Bootstrap Lifecycle Model 版本与 CENTRE Runtime 版本解耦。Bootstrap 模式的增删属于 Protocol 层变更，需 RFC Process。

---

## 实现

### Protocol 层

- `constitution/constitution/contracts/runtime-contract.md` §2.6 Bootstrap Interface — 定义 Bootstrap 生命周期

### Runtime 层

- `Skills/aise-bootstrap/SKILL.md` — Bootstrap Engine 实现（BIRTH / RESTORE / RECOVERY）
- `Skills/handoff/SKILL.md` — RESUME R3.5 调用 `aise-bootstrap.restore()`
- `Skills/aise-admission/SKILL.md` — 消费 Bootstrap State

### 未来扩展

当 Tenant Identity Layer 引入后，Bootstrap 可能需要从 runtime-contract 中抽离为独立的 `bootstrap-contract.md`，因为 Bootstrap 将被 Runtime、Factory、Installer、Tenant Layer 共同依赖。

---

## 版本

- RFC-0011: v1.0 FROZEN
- CENTRE Protocol: 2.0.0-frozen
- CENTRE Foundation: v3.2.0