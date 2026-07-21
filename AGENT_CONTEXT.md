# AGENT_CONTEXT.md — Bootstrap Contract

> Repository: aise-standard
> Role: Protocol Authority
> Version: 2.0.0-frozen
> **Entry Point: AGENTS.md (read first for behavior rules)**

---

## 0. Bootstrap Model

```
Agent Arrival
      │
      ▼
AGENTS.md          ← Behavior Rules (what you can/cannot do)
      │
      ▼
AGENT_CONTEXT.md   ← World Model (where you are, what is real)
      │
      ▼
Repository Reality
```

---

## 1. Repository Identity

- **逻辑身份**：CENTRE Protocol Specification
- **角色**：Protocol Authority — 定义 Constitution、Contracts、Schemas、Governance Rules
- **协议版本**：2.0.0-frozen
- **Foundation**：CENTRE v3.2.0

---

## 2. Authority Ranking

Agent 读取本仓库时，按以下优先级：

| 优先级 | 文件 | 说明 |
|--------|------|------|
| 1 | AGENT_CONTEXT.md | 身份契约声明（最高权威） |
| 2 | PROJECT_STATE.json | 项目状态 |
| 3 | VERSION | 当前协议版本 |
| 4 | .project/centre.protocol.json | CENTRE Protocol Manifest |
| 5 | protocol/ | 协议定义（Constitution、Contracts、Schemas） |
| 6 | constitution/ | Constitution 文档（v1.0.0frozen，独立 artifact） |
| 7 | README.md | 人类入口 |

---

## 3. Forbidden Context

当前项目上下文外的历史文档不纳入 Agent 读取范围。

---

## 4. Authority Boundaries

**本仓库拥有**：
- Protocol 定义权
- Constitution 定义权
- Contract 定义权
- Schema 定义权
- Governance Rules 定义权

**本仓库不拥有**：
- Runtime 执行（属于 aos-runtime）
- Artifact 构建（属于 aos-factory）
- 部署安装（属于 aos-installer）
- Agent 状态（属于 Agent Instance）
- Project 上下文（属于 Project）

---

## 5. 相关仓库

| 仓库 | 角色 | 关系 |
|------|------|------|
| aise-standard | Protocol Authority | 本仓库 |
| aos-runtime | Execution Layer | 消费 Protocol |
| aos-factory-new | Build Layer | 消费 Protocol Contracts |
| aos-installer | Deploy Layer | 消费 Protocol Artifact |

---

## 6. 版本

- AGENT_CONTEXT.md: v1.0
- Protocol: 2.0.0-frozen
- CENTRE Foundation: v3.2.0