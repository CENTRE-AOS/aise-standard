# AOS Protocol Artifact

> 版本: 1.0
> 状态: Frozen
> 生效日期: 2026-07-17
> 所属: AOS Architecture Constitution

---

## Protocol Artifact 是什么

Protocol Artifact 是 AOS Foundation 从 frozen tag 生成的不可变协议产品。它脱离 Git 仓库独立存在，是任何环境安装 AOS 协议的"宪法文本"。

## 身份定义

```json
{
  "protocol_artifact": {
    "name": "AISE Protocol",
    "version": "2.6.0",
    "edition": "frozen",
    "canonical_identity": "aise-protocol-v2.6.0",
    "immutable": true,
    "produced_by": {
      "specification": "aos-protocol-factory",
      "runtime": "aos-runtime"
    },
    "protocol_version": "1.0",
    "released_at": "2026-07-17"
  }
}
```

## Artifact 包结构

```
aise-protocol-v2.6.0/
├── manifest.json              ← 协议身份声明
├── CONSTITUTION.md            ← 架构宪章
├── NAMING.md                  ← 命名体系
├── identity/
│   ├── protocol.md
│   └── schema.json
├── structure/
│   ├── protocol.md
│   ├── schema.json
│   └── aos-home.md            ← AOS_HOME 规范
├── state/
│   ├── protocol.md
│   └── schema.json
├── evolution/
│   ├── protocol.md
│   ├── schema.json
│   ├── git-policy.md          ← Git 治理规则
│   ├── lifecycle.md           ← 生命周期
│   └── runtime-boundary.md    ← Runtime 边界
├── hook-templates/            ← 薄适配器模板
│   ├── pre-commit.ps1
│   ├── commit-msg.ps1
│   ├── pre-push.ps1
│   └── post-merge.ps1
└── templates/                 ← 项目模板
    ├── PROJECT_BLUEPRINT.template.md
    ├── CHANGELOG.template.md
    └── HANDOFF.template.md
```

## 关键边界

- Protocol Artifact 中**没有任何 `.ps1`、`.py`、`.exe`**（hook-templates/ 中是薄适配器模板，不含规则，规则在 evolution/git-policy.md）
- Artifact 是**不可变**的 — 一旦发布，永不修改
- Artifact 的**身份独立于生产仓库** — 不包含仓库路径、不包含开发环境信息

## 生命周期

```
ACP+AISE Factory (develop)
    │
    │ 研发、测试、迭代
    ▼
ACP+AISE Factory (main)
    │
    │ aise-archive
    ▼
vX.Y.Zfrozen tag (immutable)
    │
    │ aise-release
    ▼
Protocol Artifact Package (.zip)
    │
    │ aise-install
    ▼
AOS_HOME/protocol/aise/vX.Y.Z/
    │
    │ current → vX.Y.Z
    ▼
Project 声明使用
```

## 版本号规则

| 场景 | 版本号 | 说明 |
|------|--------|------|
| 协议语义变更 | `v2.6.0frozen` | 新协议版本 |
| Bug 修复 | `v2.6.1frozen` | 新补丁版本 |
| 紧急热修复 | `v2.6.0frozen-patch-N` | 标记性补丁，不覆盖原 tag |

## Frozen Tag 不可变原则

- `vX.Y.Zfrozen` tag 一旦推送到远程，**永不移动**
- 任何修改必须通过**新版本号**发布
- 如果发生 tag 被 force-move，必须记录 Exception Record

## 安装后的环境

```
AOS_HOME/
├── protocol/
│   └── aise/
│       ├── v2.5.0/            ← 已安装的历史版本
│       ├── v2.6.0/            ← 当前版本
│       └── current → v2.6.0/  ← 符号链接
├── runtime/
├── adapters/
├── registry.json
├── constitution/
└── cache/
```