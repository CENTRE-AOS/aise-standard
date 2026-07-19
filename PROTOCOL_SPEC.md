# AISE Protocol Specification

## Overview

AISE（Agent Software Engineering Protocol）是 AOS 生态的一号协议，定义了 Agent 与项目交互的工程契约。

## Protocol Architecture

```
AISE Standard
    │
    │ defines validity
    ▼
CENTRE Runtime
    │
    │ Runtime Adapter Registry
    │
    │ execution lifecycle
    ▼
External Environment
```

## Core Contracts

### Bootstrap Contract
- **Define**: Protocol 定义 Bootstrap 规则
- **Execute**: Runtime 执行 Bootstrap 流程
- **Validate**: Factory 验证 Bootstrap 合规

### Artifact Contract
- **Manifest**: 身份声明
- **Hash**: 内容完整性
- **Signature**: 来源认证
- **Compatibility**: 兼容性声明

### Compatibility Contract
- Semantic version range declaration
- Artifact Owner declares compatibility
- Consumer matches against declaration

## Protocol Layers

| Layer | Directory | Description |
|-------|-----------|-------------|
| Identity | `protocol/identity/` | Agent 身份协议 |
| State | `protocol/state/` | 状态管理协议 |
| Structure | `protocol/structure/` | AOS_HOME 结构协议 |
| Evolution | `protocol/evolution/` | 演进与版本协议 |
| Governance | `protocol/governance/` | 治理与合规协议 |

## Version

Current: 2.0.0-frozen