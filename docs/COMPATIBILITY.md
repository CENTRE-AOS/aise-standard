# AISE Compatibility Matrix

> 版本：v3.1.0
> 状态：Active
> 日期：2026-07-19

## Current Release

| 仓库 | 版本 | 状态 | 说明 |
|------|------|------|------|
| **agent-governance** | v3.1.0 | Active | CENTRE AOS Runtime v0.1 — 参考运行时 |
| **aise-standard** | v2.6.0frozen | Frozen | AISE Protocol 1.0 规范（Protocol Factory） |

## Supported Range

```
agent-governance v3.1.x
    │
    └── supports aise-standard: >=2.6.0frozen, <3.0.0

| agent-governance | aise-standard | 兼容性 |
|------------------|---------------|--------|
| v3.1.x | v2.6.0frozen | 完全兼容 |
| v2.5.x | v2.5.0frozen | 冻结（不再更新） |
| v3.0.x | v2.x.x | 不兼容（需 Migration） |
```

## Upgrade Rule

1. **AISE Protocol major/minor upgrade** 必须通过 Migration Protocol（`aise-standard/Policies/upgrade-policy.md`）
2. **Project AISE snapshot** 不自动升级，由项目显式决定
3. **agent-governance** 升级后，参考模板同步更新，但不强制项目升级
4. **跨大版本升级**（如 v1.x → v2.x）必须先升级到 v1.x 最新版

## Migration Path

```
v1.5.x → v2.5.0frozen → v2.6.0frozen
  │
  └── Migration: runtime/commands/migrations/migration.ps1
      说明: Runtime 目录重构，scripts/ → runtime/commands/
```

## Project AISE Snapshot Rule

```
项目 .agent-entry.json protocol version = 1.0
         │
         ├── agent-governance 升级到 v3.1.0
         │   └── 项目不自动升级
         │
         ├── 用户显式执行 aise migrate
         │   └── 项目升级到运行时 v3.1.0
         │
         └── 检测到版本不兼容
             └── Agent 提示用户执行 Migration
```

## 版本检测流程

```
Agent 进入项目
    ↓
读取 .agent-entry.json
    │   └── protocol = "AISE", version = "1.0"
    ↓
读取 Governance Runtime 的 VERSION
    │   └── agent-governance v3.1.0
    ↓
读取 COMPATIBILITY.md
    ↓
检查 agent-governance 与项目声明的 protocol_version 是否兼容
    ↓
    ├── 兼容 → 继续
    ├── 向后兼容 → 警告 + 建议升级
    └── 不兼容 → 阻止 + 提示 Migration
```

> 注意：项目本身不携带 `aise-standard/` 目录。协议规范由独立的 `aise-standard` 仓库集中维护；项目仅通过 `.agent-entry.json` 声明遵循的 AISE Protocol 版本。