# Structure Protocol

> **版本**: 1.0
> **状态**: Frozen
> **所属协议族**: AISE Protocol 1.0

## 1. 目的

回答 Agent 进入项目时的一个问题：

> **How is the project organized?**

Structure Protocol 定义项目如何被组织——目录结构、资产布局、文档规范。Agent 通过 Structure Protocol 快速理解项目骨架，不需要扫描全部文件。

## 2. 核心原则

- 项目资产属于项目，不属于任何 Agent。
- 所有 Agent 读写同一套资产。
- 资产必须被版本控制（Git）。
- Agent 私有 Memory 不能成为事实来源。
- 项目结构是描述性的，不是配置驱动的。

## 3. 项目结构

```text
Project/
├── .agent-entry.json          # Protocol Manifest（Identity Protocol）
├── PROJECT_BLUEPRINT.md        # 项目蓝图
├── CHANGELOG.md                # 版本历史
├── .project/                   # 项目级知识资产
│   ├── aos.json                # AOS 协议声明
│   ├── state/                  # 当前状态（State Protocol）
│   ├── memory/                 # 项目级知识
│   │   ├── knowledge/
│   │   ├── patterns/
│   │   └── glossary/
│   ├── decisions/              # ADR 架构决策记录
│   ├── architecture/           # 架构文档
│   ├── journal/                # Agent 活动日志
│   └── audit/                  # 合规审计记录
├── src/                        # 业务代码
├── tests/                      # 测试
├── docs/                       # 文档
└── config/                     # 配置
```

## 4. 核心资产文件

| 文件 | 类型 | 说明 |
|------|------|------|
| `.agent-entry.json` | 身份 | 项目协议声明 |
| `PROJECT_BLUEPRINT.md` | 蓝图 | 项目架构、目标、状态 |
| `CHANGELOG.md` | 历史 | 版本变更历史 |
| `.project/aos.json` | 声明 | AOS 协议版本声明 |
| `.project/state/` | 状态 | 当前项目状态（State Protocol） |
| `.project/memory/` | 知识 | 项目级知识资产 |
| `.project/decisions/` | 决策 | ADR 架构决策记录 |

## 5. 读写规则

- **允许写入**: `.project/` 内文件、业务代码（按 Mission Boundary）
- **禁止写入**: `.agent-entry.json`、`.git/`、`secrets/.env`
- **读取**: 所有 Agent 均可读取 `.project/` 和 `PROJECT_BLUEPRINT.md`

## 6. 与 State Protocol 的关系

Structure 是项目的骨架，State 是项目的当前状态。

```text
Structure (静态组织)
   ↓
State (动态状态)
```

## 7. Schema

参见: `schema.json`