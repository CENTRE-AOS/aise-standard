# aise-standard

aise-standard 是 **AISE（Agent Software Engineering Protocol）** 的协议规范仓库。

## 角色

**Protocol Authority** — 定义 AOS 生态的协议规则、Schema、合约和治理框架。

## 职责

- 定义 Protocol Contracts（Bootstrap Contract、Artifact Contract、Compatibility Contract）
- 维护 Protocol Schema（identity、state、structure、evolution、governance）
- 发布 Protocol Registry（compliance、routing、skills、version）
- 管理 Constitution（Foundation Freeze、SKILL_ARCHITECTURE）
- 提供 Project Bootstrap Templates

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
│   │   └── handoff/    # 交接协议
│   └── contracts/      # 工程合约
├── constitution/       # 宪法文档
├── registry/           # 协议注册表
├── templates/          # 项目引导模板
├── docs/               # 文档
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
- Repository extracted from legacy-monorepo-freeze-v3.1.0

## 治理

本仓库由 AISE Protocol Authority 治理。所有 Protocol 修改需遵循 RFC Process。