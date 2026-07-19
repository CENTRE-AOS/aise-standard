# AISE Protocol Mapping

> 版本: 1.0
> 状态: Frozen
> 所属协议族: AISE Protocol 1.0

## 协议总览

```text
AISE Protocol
├── CONSTITUTION.md          # 架构宪章
├── NAMING.md                # 命名体系
├── AOS_POSITIONING.md       # AOS 定位
├── ARTIFACT.md              # Protocol Artifact 定义
├── Identity Protocol        # 我是谁
├── Structure Protocol       # 项目如何组织
├── State Protocol           # 当前状态
└── Evolution Protocol       # 如何演进
```

## 协议职责矩阵

| 协议 | 问题 | 核心概念 | 对应命令 |
|------|------|---------|---------|
| **Identity** | Who am I? | .agent-entry.json, project_id, protocol | aise-bootstrap |
| **Structure** | How is the project organized? | .project/, PROJECT_BLUEPRINT.md, aos.json | aise-init |
| **State** | What is the current state? | handoff, mission, timeline, lifecycle | agent_exit, handoff |
| **Evolution** | How does the project evolve? | git-policy, lifecycle, frozen, release | aise-verify, aise-archive, aise-release, aise-install |

## Agent 读取顺序

```text
1. Identity   → .agent-entry.json          → 我是谁
2. State      → .project/state/            → 当前状态
3. Blueprint  → PROJECT_BLUEPRINT.md       → 架构全貌
4. Decision   → .project/decisions/        → 为什么这样设计
5. Changelog  → CHANGELOG.md               → 最近变化
6. Git        → branch/tag/commit          → 真实快照
7. Code       → src/                        → 开始工作
```

## 协议版本历史

| 协议版本 | AISE 版本 | 变更 |
|---------|----------|------|
| 1.0 | v2.6.0frozen | 四协议语义重命名：Asset→Structure, Context→State, Governance→Evolution。新增 CONSTITUTION、NAMING、AOS_POSITIONING、ARTIFACT。 |
| 1.0 | v2.5.0frozen | 原始四协议：Identity, Asset, Context, Governance |