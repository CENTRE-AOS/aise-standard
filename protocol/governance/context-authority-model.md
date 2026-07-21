# Context Authority Model

> RFC-0010 | Context Authority | v1.0
> Status: Foundation Candidate
> Part of: CENTRE Protocol — Bootstrap Contract
> Freeze Level: Level 1 (Contract — RFC required for modification)

---

## 1. 定位

Context Authority Model 定义了 Agent 进入任何 CENTRE 治理仓库后，如何区分不同来源的信息的真实性。

**核心原则**：

```
Agent 不自己判断真相。
系统向 Agent 描述真相。
```

---

## 2. 双层 Bootstrap 模型

Agent 进入仓库后，按以下顺序建立上下文：

```
Agent Arrival
        │
        ▼
┌─────────────────────────────────┐
│ Layer 1: AGENTS.md              │
│   Universal Agent Instruction   │
│   回答："如何工作？"              │
│                                 │
│   受众：所有 Agent 平台           │
│   内容：行为规则、读取顺序、       │
│         禁止操作、权限边界        │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ Layer 2: AGENT_CONTEXT.md       │
│   CENTRE Identity Contract      │
│   回答："这是哪？"                │
│                                 │
│   受众：CENTRE 治理体系           │
│   内容：身份声明、Authority       │
│         Ranking、Forbidden      │
│         Context、权限边界        │
└────────────┬────────────────────┘
             │
             ▼
       Repository Reality
```

---

## 3. Authority Level 体系

### 3.1 五级权威模型

| Level | 名称 | 来源 | 可修改性 | 示例 |
|-------|------|------|---------|------|
| 0 | Protocol Authority | CENTRE Protocol | NEVER | Constitution, Contracts, Freeze Documents |
| 1 | Repository Identity | PROJECT_STATE, AGENT_CONTEXT | Identity fields only | 仓库角色、权限边界 |
| 2 | Architecture Truth | PROJECT_BLUEPRINT, RFC | RFC required | 架构文档、设计决策 |
| 3 | Implementation | src/, runtime/, tests/ | Allowed | 代码、配置、测试 |
| 4 | Historical Reference | archive/, migration/, .project/memory/ | Read-only | 历史快照、迁移记录 |

### 3.2 冲突解决

当不同 Level 的信息冲突时：

```
Level 0 > Level 1 > Level 2 > Level 3 > Level 4

Level 0 (Protocol) 的定义永远覆盖 Level 3 (Implementation) 的实现。
Level 4 (Historical) 不能作为 Level 2 (Architecture) 的当前状态。
```

---

## 4. 仓库必须文件

每个 CENTRE 治理仓库 **MUST** 包含：

| 文件 | 层级 | 必须 |
|------|------|:--:|
| AGENTS.md | Bootstrap Layer 1 | ✅ |
| AGENT_CONTEXT.md | Bootstrap Layer 2 | ✅ |
| PROJECT_STATE.json | Authority Level 1 | ✅ |
| VERSION | Authority Level 1 | ✅ |
| PROJECT_BLUEPRINT.md | Authority Level 2 | ✅ |
| CHANGELOG.md | Authority Level 2 | ✅ |

---

## 5. Forbidden Context 声明

### 5.1 必须标记为 Historical Reference 的内容

| 路径模式 | 说明 |
|---------|------|
| `ARCHIVE-*.md` | 历史快照 |
| `.project/memory/` | 项目记忆（历史知识） |
| `.project/decisions/` | ADR（历史决策） |
| `.project/journal/` | Agent 活动日志 |
| `.handoff/` | 交接快照 |
| `.sync/` | 环境同步状态 |
| `migration/` | 迁移过程文件 |
| `docs/*-proposal*.md` | 历史提案 |
| `docs/*-plan*.md` | 历史计划 |

### 5.2 标记规则

在 AGENT_CONTEXT.md 的 Forbidden Context 节中声明。Agent MUST NOT 将 Historical Reference 作为当前系统上下文。

---

## 6. 身份声明规则

### 6.1 逻辑身份 vs 物理身份

当仓库的物理名称与逻辑身份不同时，AGENT_CONTEXT.md **MUST** 声明两者：

```
物理身份：agent-governance (GitHub repository name)
逻辑身份：aos-runtime (CENTRE Runtime)
```

Agent MUST 以逻辑身份为准。

### 6.2 版本独立性

不同 artifact 的版本号独立：

| Artifact | 版本 | 说明 |
|----------|------|------|
| CENTRE Foundation | v3.2.0 | 架构兼容性边界 |
| Protocol | 2.0.0-frozen | 协议版本 |
| Constitution | 1.0.0frozen | 宪法文档版本（独立 artifact） |
| Runtime | 3.2.0 | 执行引擎版本 |
| Installer | 1.1.0 | 部署层版本 |
| Factory | 1.1.0 | 构建系统版本 |

Agent MUST NOT 假设所有版本号相同。

---

## 7. 版本

- Context Authority Model: v1.0
- CENTRE Protocol: 2.0.0-frozen
- CENTRE Foundation: v3.2.0
- Freeze Level: Level 1 (Contract)