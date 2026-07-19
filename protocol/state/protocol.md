# State Protocol

> **版本**: 1.0
> **状态**: Frozen
> **所属协议族**: AISE Protocol 1.0

## 1. 目的

回答 Agent 进入项目时的问题：

> **What is the current state?**

State Protocol 定义项目当前状态如何被记录、恢复和重建。Agent 通过 State Protocol 在不扫描项目的情况下，快速知道"项目现在是什么状态"。

## 2. 核心原则

- State 是 Structure 的动态状态。
- 项目状态由 Handoff、Mission、Timeline 组成。
- 所有 Agent 基于同一 State 工作。
- State 不依赖 Event Sourcing —— Handoff + Git + Changelog + Decisions 已足够。

## 3. State 结构

```text
.project/state/
├── handoff/              # Handoff 交接记录（当前施工现场说明牌）
│   └── latest.json
├── mission.json          # 当前任务/目标
└── timeline.jsonl        # 规范事件记录
```

## 4. 项目生命周期状态机

```
Create ──→ Bootstrap ──→ Develop ──→ Checkpoint ──→ Freeze ──→ Handoff ──→ Continue
  │           │            │            │              │           │
  │           │            │            │              │           │
aise-init  aise-bootstrap  git flow   aise-verify   aise-archive  agent_exit
                                                │
                                          vX.Y.Zfrozen
                                          (immutable tag)
```

## 5. Handoff 是施工现场说明牌

Handoff 不是 Event Schema，而是项目当前状态的快照：

```json
{
  "snapshot": {
    "version": "v2.6.0frozen",
    "branch": "develop",
    "commit": "abc123"
  },
  "change": {
    "type": "architecture",
    "scope": ["protocol", "runtime"],
    "reason": "AOS architecture realignment"
  },
  "verification": {
    "aise-verify": "pass",
    "aise-authority": "pass"
  },
  "status": "ready for next phase"
}
```

## 6. Agent 恢复流程

```text
1. 读取 .project/aos.json → 确认协议版本
2. 读取 .project/state/handoff/latest.json → 知道当前状态
3. 读取 .project/state/mission.json → 知道当前任务
4. 读取 PROJECT_BLUEPRINT.md → 知道架构
5. 读取 CHANGELOG.md → 知道最近变化
6. 读取 Git (branch/tag/commit) → 确认真实快照
7. 开始工作
```

## 7. Schema

参见: `schema.json`