# CENTRE Authority Matrix

> Version: 1.1.0-frozen
> Date: 2026-07-21
> Phase: S0.6 — Remote Ownership Freeze
> Status: FROZEN

---

## 1. 五层 Authority 定义

| Level | Authority | Repository | Owner | Visibility | Status | 输入 | 输出 | Consumer |
|:-----:|-----------|-----------|-------|:----------:|:------:|------|------|----------|
| **A0** | Protocol Authority | aise-standard | `<owner>` | public | ACTIVE | — | Protocol Contracts | A2 Factory |
| **A1** | Source Authority | aos-runtime | `<owner>` | public | ACTIVE | A0 Protocol | Source Code (read-only) | A2 Factory |
| **A2** | Build Authority | aos-factory | `<owner>` | private | ACTIVE | A0 Protocol + A1 Source | Signed Artifact | A3 Registry |
| **A3** | Artifact Authority | artifact-registry | `<owner>` | TBD | **RESERVED** | A2 Signed Artifact | Published Artifact | A4 Installer |
| **A4** | Install Authority | aos-installer | `<owner>` | private | ACTIVE | A3 Published Artifact | Installed Artifact | Runtime Instance |

## 2. A3 Reserved Declaration

A3 Artifact Authority 当前未实现（Phase 3），但必须在模型中预留。

**禁止**: A2 Factory 同时承担 A3 Artifact 职责。Factory 创建 Artifact，但不得管理 Artifact 的分发和存储。

**禁止**: A4 Installer 直接消费 A2 Factory 产物。必须经过 A3 Registry。

A3 预留属性：

```json
{
  "authority": "artifact",
  "authority_level": "A3",
  "status": "RESERVED",
  "repository": "artifact-registry",
  "planned_phase": "Phase 3.0",
  "input": "A2 Signed Artifact",
  "output": "Published Artifact",
  "consumer": ["aos-installer"]
}
```

## 3. Authority 不可混合规则

| 规则 | 说明 |
|------|------|
| R1 | 没有 Authority 可以执行两个连续阶段 |
| R2 | A0 定义规则，不执行规则 |
| R3 | A1 提供代码，不构建代码 |
| R4 | A2 构建产物，不安装产物 |
| R5 | A3 存储分发，不部署执行 |
| R6 | A4 部署引导，不运行执行 |

## 4. 供应链数据流

```
A0 Protocol ──Contract──▶ A2 Factory
A1 Source   ──Read-Only─▶ A2 Factory
A2 Factory  ──Artifact──▶ A3 Registry (Reserved)
A3 Registry ──Published─▶ A4 Installer
A4 Installer──Installed─▶ Runtime Instance
```

## 5. 禁止路径

```
FORBIDDEN:
  A2 → A4 (Factory 直接交付给 Installer)
  A1 → A4 (Installer 直接读取 Source)
  A2 managing A3 (Factory 管理 Artifact 分发)
  A0 → A4 (Installer 直接读取 Protocol)
```

## 6. Repository Naming

| 本地目录 | GitHub 仓库名 | 冻结状态 |
|---------|-------------|:------:|
| aise-standard | aise-standard | FROZEN |
| aos-runtime | aos-runtime | FROZEN |
| aos-factory-new | aos-factory | **RENAME** (本地目录保留，GitHub 使用 aos-factory) |
| aos-installer | aos-installer | FROZEN |
| aos-migration | aos-migration | FROZEN (Archive) |

**注意**: `aos-factory-new` 是迁移过渡名。GitHub 仓库名使用 `aos-factory`。本地目录名后续可重命名，不影响 Git 历史。