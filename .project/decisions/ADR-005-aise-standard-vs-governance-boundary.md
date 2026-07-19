# ADR-005: aise-standard vs agent-governance 职责边界

## Status

Accepted — 2026-07-16

## Context

随着 AISE v1.2 定位为 Protocol Freeze，需要明确 `aise-standard` 与 `agent-governance` 两个仓库的职责边界，避免标准与运行时混放、双写、职责重叠。

## Decision

| 仓库 | 定位 | 允许内容 | 禁止内容 |
|------|------|---------|---------|
| `aise-standard` | Frozen Standard | Contract、Policy、Skill 定义、Template、Schema、Registry | 脚本、安装逻辑、治理运行时、项目数据 |
| `agent-governance` | Governance Runtime | Bootstrap、Install、Audit、Update、Migration、Validation、Repository Governance | 复制 Standard 内容、存储项目 Memory Primary |

## Consequences

- `aise-standard/Git-Governance/` 仅保留声明性规则（git-gate.md、policies.json）。
- Hook 脚本、命令脚本、迁移脚本全部移入 `agent-governance/scripts/`。
- `agent-governance/aise-template/AISE/` 复制内容删除，改为运行时引用 `aise-standard`。
- 项目 Memory / Journal / Handoff Primary 保留在项目本地，Governance 只保留索引/Cache。

## Related Documents

- `docs/AISE-v1.2-Architecture-Proposal.md`
- `docs/AISE-Governance-Refactoring-Plan.md`
- `AISE/Policies/memory-ownership.md`
