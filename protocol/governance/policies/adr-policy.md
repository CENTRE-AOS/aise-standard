# Architecture Decision Record (ADR) Policy

> 版本：v1.1.0
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

ADR 记录项目中的关键架构决策。当 Agent 做出影响项目长期结构的选择时，必须记录决策原因，确保未来接替的 Agent 理解"为什么"。

## 2. 触发条件

以下场景必须创建 ADR：

- 技术选型（数据库、框架、语言、库）
- 架构调整（分层、模块拆分、接口变更）
- 模式选择（设计模式、数据流模式、部署模式）
- 放弃的方案（为什么选了 A 而不是 B）

## 3. ADR 格式

位置：`.project/decisions/ADR-<NNN>-<slug>.md`

```markdown
# ADR-001: 数据库选择

- **日期**: 2026-07-14
- **状态**: [已采纳 / 已替代 / 已废弃]
- **Agent**: DeepSeek-V4-Pro
- **Mission**: 初始化项目数据层

## 背景

项目需要持久化存储，需要在 SQLite 和 PostgreSQL 之间选择。

## 决策

采用 PostgreSQL。

## 原因

1. SQLite 无法满足多 Agent 并发写入需求
2. PostgreSQL 支持事务隔离级别，适合 Autonomous Agent 模式
3. 项目目标规模需要完整 SQL 支持

## 排除的方案

### MongoDB
- 原因：事务模型不符合需求，项目需要关系型数据
- 代价：放弃灵活的文档模型

### SQLite  
- 原因：不支持并发写入，不适合多 Agent 环境
- 代价：放弃零配置部署的便利性

## 影响

- 需要 PostgreSQL 运行环境
- 部署脚本需要包含数据库初始化
- 测试环境需要配置测试数据库

## 替代者

（如果是被替代的 ADR，填写替代它的 ADR 编号）
```

## 4. ADR 状态流转

```
proposed → accepted → superseded / deprecated
```

| 状态 | 说明 |
|------|------|
| proposed | 提案中，待讨论 |
| accepted | 已采纳，当前生效 |
| superseded | 已被新 ADR 替代 |
| deprecated | 已废弃，不再使用 |

## 5. 索引文件

`.project/decisions/INDEX.md` 维护所有 ADR 的快速索引：

```markdown
# ADR Index

| 编号 | 标题 | 状态 | 日期 |
|------|------|------|------|
| ADR-001 | 数据库选择 | accepted | 2026-07-14 |
| ADR-002 | 认证方案 | superseded by ADR-005 | 2026-07-14 |
```

## 6. 接替时读取

新 Agent 接替时，按以下顺序理解项目决策：

1. `PROJECT_BLUEPRINT.md` — 架构全景
2. `.project/decisions/INDEX.md` — 决策索引
3. 每个 `ADR-*.md` — 决策详情
4. `.project/audit/` — 治理事件

## 7. 变更控制

- ADR 创建后不可删除（可标记为 deprecated）
- 替代 ADR 必须在"排除的方案"中引用旧 ADR