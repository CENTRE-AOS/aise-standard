# AISE Standard — 项目启动说明书

> 消费者: Human, Developer, Agent
> 本文件描述项目的基本信息、依赖和启动方式。
> Bootstrap 流程由 CENTRE Runtime 的 aise-bootstrap Engine 统一执行，参见 `.agent/bootstrap.md`。

## 项目概述

aise-standard 是 CENTRE Protocol 的权威定义者（A0 Protocol Definition）。负责定义 Protocol Contracts、Constitution、Registry 和 Artifact Schema。

## 依赖

- CENTRE Protocol 2.0.0-frozen（自身定义）
- 无外部运行时依赖

## 使用的 Contracts

本项目定义以下 Contracts（位于 `constitution/constitution/contracts/`）：

- artifact-contract.md
- build-contract.md
- installer-contract.md
- artifact-consumption-contract.md
- runtime-contract.md
- skill-contract.md
- event-contract.md

## 项目结构

```
aise-standard/
├── constitution/          # Constitution + Contracts
├── protocol/              # Protocol 定义
├── registry/              # Protocol Registry
├── docs/                  # 文档
├── .agent/                # Agent 元数据
├── AGENTS.md              # 行为规则
├── AGENT_CONTEXT.md       # 身份契约
├── PROJECT_CONTEXT.md     # 认知边界
├── PROJECT_BLUEPRINT.md   # 架构蓝图
├── BOOTSTRAP.md           # 本文件
├── CHANGELOG.md           # 版本历史
└── VERSION                # 版本号
```

## 开发说明

- Protocol 修改需通过 RFC 流程
- Constitution 修改需通过 Freeze 流程
- 冻结的 Contract 不可直接修改