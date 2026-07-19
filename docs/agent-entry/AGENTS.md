# AI Agent Entry Protocol

> WARNING: This repository implements AISE Protocol.
> Before any analysis or modification:
>
> 1. Locate `.agent-entry.json` in project root
> 2. Read `.agent-entry.json` to confirm protocol version and governance provider
> 3. Read `PROJECT_BLUEPRINT.md` and `CHANGELOG.md`
> 4. Load `.project/memory/` and `.project/context/` if present
> 5. Execute `aise bootstrap` via the declared governance provider if available
>
> **Do not use default agent behavior before completing bootstrap.**

---

# Repository Agent Rules

本项目遵循 **AISE Protocol 1.0 / aise-standard v2.5.0frozen**。

## 进入协议

进入本项目的 AI Agent 必须首先读取：

1. `.agent-entry.json` — Protocol Manifest
2. `PROJECT_BLUEPRINT.md` — 项目蓝图
3. `CHANGELOG.md` — 版本历史
4. `.project/memory/` — 项目记忆（如存在）
5. `.project/context/` — 项目上下文（如存在）

**未完成读取前禁止修改任何文件。**

## 行为约束

所有行为受 AISE Contracts 约束：

- `Contracts/engineering-contract.md`
- `Contracts/repository-contract.md`
- `Contracts/metadata-contract.md`
- `Contracts/handoff-protocol.md`

## 安全规则

所有操作必须遵循 `Policies/security-policy.md` 中的 C1-C9 安全元规则。

失败即触发 `Policies/rollback-policy.md` 中的 C3 Recovery Protocol。

## 模式

本项目采用 **Autonomous Agent Engineering** 模式。
AI 拥有全部工程文件权限，受 Contract 与用户 Mission 约束。
