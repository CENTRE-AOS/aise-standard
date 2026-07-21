# aise-standard

aise-standard 是 **CENTRE Protocol** 的协议规范仓库。

## 角色

**Protocol Authority** — 定义 AOS 生态的协议规则、Schema、合约和治理框架。

## 职责

- 定义 Protocol Contracts（Bootstrap Contract、Artifact Contract、Compatibility Contract）
- 维护 Protocol Schema（identity、state、structure、evolution、governance）
- 发布 Protocol Registry（compliance、routing、skills、version）
- 管理 Constitution（Foundation Freeze、SKILL_ARCHITECTURE）
- 提供 Project Bootstrap Templates
- 定义 Agent Understanding Layer（Bootstrap Contract: AGENTS.md + AGENT_CONTEXT.md + Context Authority Model）

## Agent 入口

Agent 进入本仓库时，遵循双层 Bootstrap 模型：

1. **AGENTS.md** — Universal Agent Instruction（行为规则、读取顺序、禁止操作、权限边界）
2. **AGENT_CONTEXT.md** — CENTRE Identity Contract（身份声明、Authority Ranking、Forbidden Context）

详见 `protocol/governance/context-authority-model.md`（RFC-0010）。

## 仓库边界

```
aise-standard/
├── protocol/           # 协议定义
│   ├── identity/       # 身份协议
│   ├── state/          # 状态协议
│   ├── structure/      # 结构协议
│   ├── evolution/      # 演进协议
│   ├── governance/     # 治理协议
│   │   ├── policies/   # 策略定义
│   │   ├── handoff/    # 交接协议
│   │   └── context-authority-model.md  # 五级权威体系
│   └── contracts/      # 工程合约
├── constitution/       # 宪法文档
├── registry/           # 协议注册表
├── templates/          # 项目引导模板
├── docs/               # 文档
├── AGENTS.md           # 通用 Agent 行为规则
├── AGENT_CONTEXT.md    # CENTRE 身份契约
└── .project/           # 项目资产（decisions, memory）
```

## 与其他仓库的关系

| 仓库 | 关系 |
|------|------|
| aos-runtime | aise-standard 定义 Protocol，aos-runtime 执行 Protocol |
| aos-factory | aise-standard 定义 Schema，aos-factory 验证 Schema |
| aos-installer | aise-standard 定义 Bootstrap Contract，aos-installer 实现 Bootstrap |
| user-project | aise-standard 提供 Templates，user-project 使用 Templates |

## 版本

- Protocol Version: 2.0.0-frozen
- CENTRE Foundation: v3.2.0

## 治理

本仓库由 CENTRE Protocol Authority 治理。所有 Protocol 修改需遵循 RFC Process。