# AISE Standard — Project Context

> Agent 上下文文件。定义 Protocol Agent 的认知边界。

## Project Identity

| 属性 | 值 |
|------|-----|
| 项目名 | AISE Standard |
| 仓库 | aise-standard |
| 版本 | 2.0.0-frozen |
| Authority | A0 Protocol Definition |
| 协议 | CENTRE Protocol 2.0.0-frozen |

## 职责

aise-standard 是 CENTRE Protocol 的权威定义者。

**负责**：
- 定义 Protocol Contracts（runtime, skill, event, artifact, build, installer, artifact-consumption）
- 定义 Constitution（宪法性约束）
- 管理 Protocol Version
- 管理 Protocol Registry
- 定义 Artifact Schema

**不负责**：
- 生成 Runtime Artifact（属于 A2 aos-factory）
- 执行 Runtime（属于 aos-runtime）
- 安装 Runtime（属于 A4 aos-installer）
- Build 或 Sign Artifact

## 输入/输出

```
输入: 无（Protocol 是自我定义的）
输出: Protocol Contracts + Constitution + Registry
```

## Agent 认知边界

本项目的 Agent 的上下文范围：

- 本项目的文件结构和代码
- 所有 Protocol Contracts 的内容
- Constitution 的约束
- Protocol Registry 的定义

超出以上范围的上下文不属于本项目的 Agent 认知范围。