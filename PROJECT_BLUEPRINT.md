# PROJECT_BLUEPRINT.md — aise-standard

> Role: Protocol Authority
> Version: 2.0.0-frozen
> CENTRE Foundation: v3.2.0
> Date: 2026-07-20

---

## 1. 身份声明

aise-standard 是 CENTRE Protocol 的唯一规范定义仓库。它定义协议规则，不执行协议。

```
CENTRE Ecosystem
      │
      ├── aise-standard (Protocol Authority)
      │     └── 定义: Constitution, Contracts, Schemas, Registry, Templates
      │
      ├── aos-runtime (Execution Layer)
      │     └── 执行: Protocol rules, Governance Loop, Skills
      │
      ├── aos-factory-new (Build Layer)
      │     └── 构建: Artifact production, validation, signing
      │
      └── aos-installer (Deployment Layer)
            └── 部署: Bootstrap, discovery, installation
```

## 2. 架构职责

### 2.1 拥有

- Protocol Constitution（宪法定义）
- Protocol Contracts（runtime-contract, skill-contract, engineering-contract, repository-contract, metadata-contract, handoff-protocol）
- Protocol Schemas（identity, state, structure, evolution）
- Protocol Registry（skills, version, compliance, git-governance）
- Project Bootstrap Templates
- Context Authority Model（RFC-0010）
- Agent Understanding Layer（AGENTS.md + AGENT_CONTEXT.md）

### 2.2 不拥有

- Protocol 执行（属于 aos-runtime）
- Artifact 构建（属于 aos-factory-new）
- 部署安装（属于 aos-installer）
- Agent 生命周期（属于 Runtime）
- Project 上下文（属于 Project）

## 3. 目录结构

```
aise-standard/
├── protocol/                    # 协议定义
│   ├── identity/                # 身份协议
│   ├── state/                   # 状态协议
│   ├── structure/               # 结构协议
│   ├── evolution/               # 演进协议
│   ├── governance/              # 治理协议
│   │   ├── policies/            # 10 策略定义
│   │   ├── handoff/             # 交接协议
│   │   └── context-authority-model.md  # 五级权威体系
│   └── contracts/               # 工程合约
├── constitution/                # 宪法文档
│   └── constitution/
│       ├── CONSTITUTION.md      # Runtime Constitution
│       └── contracts/           # Runtime & Skill Contracts
├── registry/                    # 协议注册表
│   ├── skills.json              # 14 Skill 注册
│   ├── version.json             # 版本声明
│   ├── compliance.json          # 合规配置
│   └── git-governance.json      # Git 治理配置
├── templates/                   # 项目引导模板
├── docs/                        # 文档
├── AGENTS.md                    # 通用 Agent 行为规则
├── AGENT_CONTEXT.md             # CENTRE 身份契约
├── VERSION                      # 版本号
├── CHANGELOG.md                 # 变更历史
└── .project/                    # 项目资产
```

## 4. 关键设计决策

### 4.1 Protocol Identity Evolution

AISE 1.0.0-frozen → CENTRE Protocol 2.0.0-frozen。详见 PROTOCOL_IDENTITY.md。

### 4.2 Agent Understanding Layer

新增双层 Bootstrap 模型：
- Layer 1: AGENTS.md（Universal Agent Instruction）
- Layer 2: AGENT_CONTEXT.md（CENTRE Identity Contract）

详见 protocol/governance/context-authority-model.md。

### 4.3 Authority Separation

五角色独立：
- Protocol Authority (aise-standard) — 定义规则
- Execution Authority (aos-runtime) — 执行规则
- Build Authority (aos-factory-new) — 构建 Artifact
- Deployment Authority (aos-installer) — 部署环境
- Consumer (user-project) — 消费 Artifact

## 5. 版本

- Protocol: 2.0.0-frozen
- CENTRE Foundation: v3.2.0
- Constitution: 1.0.0frozen
- Extracted from: legacy-monorepo-freeze-v3.1.0