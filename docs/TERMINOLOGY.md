# AISE 术语映射

> 版本：v3.1.0
> 状态：Active
> 来源：aise-standard v2.6.0frozen (Protocols/MAPPING.md)

## 四协议语义重命名

v2.6.0frozen 中，四协议语义进行了重命名，以更准确地反映协议职责：

| 旧名 (v2.5.0frozen) | 新名 (v2.6.0frozen) | 问题 | 说明 |
|---------------------|---------------------|------|------|
| Identity | Identity | Who am I? | 不变 |
| Asset | **Structure** | How is the project organized? | 从"资产归属"改为"组织结构" |
| Context | **State** | What is the current state? | 从"上下文"改为"当前状态" |
| Governance | **Evolution** | How does the project evolve? | 从"治理"改为"演进" |

## 迁移规则

1. **新项目** 统一使用新术语：Identity → Structure → State → Evolution
2. **旧项目** 可保留旧术语，但 Agent 应将旧名映射到新语义
3. **文档** 标注"v2.6.0frozen 重命名"字样，引用 MAPPING.md 作为权威来源
4. **Skill 实现** 逐步迁移到新术语，但必须兼容旧术语输入

## Canonical 名称

以下为 canonical 名称，Agent 实现时应优先使用：

| 协议 | Canonical 名称 |
|------|---------------|
| 协议 1 | Identity Protocol |
| 协议 2 | Structure Protocol |
| 协议 3 | State Protocol |
| 协议 4 | Evolution Protocol |

## 版本对应

| 协议版本 | AISE 版本 | 术语 |
|---------|----------|------|
| 1.0 | v2.6.0frozen | Identity / Structure / State / Evolution |
| 1.0 | v2.5.0frozen | Identity / Asset / Context / Governance (deprecated) |