# AISE Standard — Bootstrap Descriptor

> 本文件是 aise-bootstrap Skill 的项目参数描述器。
> Bootstrap Protocol 由 CENTRE Runtime 的 aise-bootstrap Engine 统一执行。
> 本文件只提供项目参数，不定义启动流程。

## Identity

- repository: aise-standard
- authority: A0 Protocol Definition
- version: 2.0.0-frozen
- protocol: CENTRE Protocol 2.0.0-frozen

## Context Scope

Agent 启动时，Bootstrap Engine 将加载以下上下文范围：

- `./` (项目根目录)
- `constitution/` (Constitution + Contracts)
- `protocol/` (Protocol 定义)
- `registry/` (Protocol Registry)

## Loading Policy

Bootstrap Engine 按以下顺序执行双层加载：

1. AGENTS.md → 行为规则
2. AGENT_CONTEXT.md → 身份契约
3. PROJECT_CONTEXT.md → 认知边界
4. PROJECT_BLUEPRINT.md → 架构真相
5. Constitution + Contracts

## Production Chain Position

- upstream: 无（Protocol 是自我定义的权威源）
- downstream: aos-factory-new（只读 Protocol Artifact）
- output: Protocol Contracts + Constitution + Registry