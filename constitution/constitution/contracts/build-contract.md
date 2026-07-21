# CENTRE Build Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-21
> 适用范围: CENTRE Foundation v3.2.0
> 前置: Artifact Contract v1.0.0, Runtime Contract v1.0.0
> 关联: build-authority-map.json, artifact-contract.md

---

## 1. 定义

CENTRE Build Contract 定义 Build Authority 的输入边界、输出边界、Pipeline 契约和禁止权力。Build Authority 是 Source → Artifact 的唯一转换路径，由 aos-factory-new 执行。

**Build 是 Source 的消费者，不是 Source 的拥有者。**

```
Source Authority (A1)
        │
        │ provides source
        ▼
Build Authority (A2)
        │
        │ produces artifact candidate
        ▼
Artifact Authority (A3)
```

## 2. Build Identity

| 属性 | 值 |
|------|-----|
| Owner | aos-factory-new |
| Consumer of | A1 Source Authority |
| Authority | BUILD_AUTHORITY |
| 职责 | 将 Source 转换为 Artifact Candidate |
| 输入 | Protocol Artifact + Runtime Source + Build Rules + Tests |
| 产出 | Artifact Candidate (.pkg, manifest, checksum) |
| 禁止 | 修改 Source、修改 Protocol、安装 Runtime、签名 Artifact |

### 2.1 Build Authority 不是什么

| 不是 | 原因 |
|------|------|
| 不是 Protocol 定义者 | Protocol 属于 aise-standard (A0) |
| 不是 Source 拥有者 | Source 属于 A1 (aos-runtime/aise-standard/aos-installer) |
| 不是 Runtime 实现者 | Runtime 源码属于 aos-runtime |
| 不是安装器 | Installer 是独立 Authority (A4) |
| 不是 Agent 管理工具 | Agent 平台由用户管理 |
| 不是 Artifact 分发者 | 分发属于 A3 Artifact Authority |

## 3. Build Input Contract

### 3.1 允许的输入

```
Build Authority 输入:

Input #1: Protocol Artifact
  - 来源: A3 Artifact Registry
  - 格式: aise-protocol-{version}.pkg
  - 验证: SHA256 checksum + signature
  - 用途: Contract Test, Compatibility Check

Input #2: Runtime Source
  - 来源: A1 Source Authority (aos-runtime)
  - 格式: Git Repository (source)
  - 验证: runtime.manifest.json 存在且有效
  - 用途: Build 主体内容

Input #3: Build Rules
  - 来源: aos-factory-new/builder/
  - 格式: build.ps1, assemble.ps1, sign.ps1
  - 用途: 定义 Build Pipeline

Input #4: Tests
  - 来源: aos-runtime/tests/
  - 格式: unit tests, contract tests, compatibility tests
  - 用途: 验证 Runtime 实现正确性
```

### 3.2 禁止的输入

```
Build Authority 禁止以下输入:

FORBIDDEN:
  Runtime Instance (已安装的 Runtime)
  User Environment Paths ($HOME, $AOS_HOME, $USERPROFILE)
  Agent Configuration
  Project Context
  Installer Source
  Secrets / Credentials
  Network Configuration
```

### 3.3 输入验证顺序

```
1. 验证 Protocol Artifact SHA256 签名
2. 验证 Runtime Source 结构完整性
3. 验证 Runtime Source 不包含 Protocol 定义（边界检查）
4. 验证 runtime.manifest.json 有效
5. 验证所有 Tests 通过
```

## 4. Build Output Contract

### 4.1 允许的输出

```
Build Authority 输出:

Output #1: Artifact Candidate
  - 格式: aos-runtime-{version}.pkg
  - 内容: runtime/* + Skills/* + system-skills/*
  - Manifest: artifact.manifest.json (遵循 artifact-contract.md §3)

Output #2: Checksum
  - 格式: aos-runtime-{version}.pkg.sha256
  - 算法: SHA256
  - 说明: 供 A3 Artifact Authority 签名前验证

Output #3: Build Report
  - 格式: build-report.json
  - 内容: timestamp, input_hashes, test_results, output_hashes
```

### 4.2 Artifact Manifest 必须遵循 artifact-contract.md

Build 生成的 `artifact.manifest.json` 必须遵循 `artifact-contract.md` §3 定义的 Schema：

```json
{
  "artifact": {
    "artifact_id": "aos-runtime",
    "artifact_version": "string",
    "artifact_type": "runtime",
    "build_time": "ISO8601",
    "origin": {
      "repository": "aos-runtime",
      "source_type": "runtime",
      "commit": "string"
    },
    "authority": {
      "level": "A2",
      "issuer": "aos-factory-new"
    },
    "checksum": {
      "algorithm": "sha256",
      "hash": "string"
    },
    "signature": {
      "algorithm": "rsa-sha256",
      "issuer": "aos-factory-new",
      "value": "string"
    }
  },
  "version_model": {
    "protocol_version": "string",
    "runtime_version": "string",
    "cli_version": "string",
    "artifact_version": "string",
    "independent": true
  },
  "compatibility": {
    "protocol_min": "string",
    "protocol_max": "string"
  },
  "bootstrap_contract": {
    "required_files": ["AGENTS.md", "AGENT_CONTEXT.md"],
    "bootstrap_contract_version": "1.0"
  },
  "immutable": true
}
```

### 4.3 禁止的输出

```
Build Authority 禁止以下输出:

FORBIDDEN:
  Modified Source (Build 不修改 Source)
  Modified Protocol (Build 不修改 Protocol)
  Installer Package (Installer 是独立 Artifact)
  Runtime Instance (Build 不安装)
  User Configuration
  Environment-specific files
```

## 5. Build Pipeline Contract

### 5.1 Pipeline 阶段（冻结）

```
Stage 1: Validate Input
  验证 Protocol Artifact 完整性和 Runtime Source 结构

Stage 2: Contract Test
  验证 Runtime 实现与 Protocol Contract 一致

Stage 3: Compatibility Check
  验证 Runtime 版本与 Protocol 版本兼容

Stage 4: Assemble
  组装 Artifact Candidate，生成 artifact.manifest.json 和 checksum

NOT INCLUDED (A3 Artifact Authority):
  Sign Artifact
  Publish Artifact
  Register in Artifact Registry
```

### 5.2 Pipeline 约束

- 4 个阶段不可跳过、不可合并
- 任何阶段失败 → 终止 Pipeline
- Build 不修改输入（Source 和 Protocol 只读）
- Build 不触发安装（只产生 Artifact Candidate）
- Build 不签名 Artifact（签名属于 A3 Artifact Authority）
- Build 不发布 Artifact（发布属于 A3 Artifact Authority）

## 6. Build Authority Forbidden Scope

### 6.0 A2/A3 Boundary Clarification (v1.0.1)

```
BUILD AUTHORITY (A2)

Produces:
    Artifact Candidate (.pkg + manifest + checksum)

Does NOT:
    Sign Artifact
    Publish Artifact
    Register in Artifact Registry
    Upload to Artifact Repository

Handoff:
    Artifact Candidate → A3 Artifact Authority


ARTIFACT AUTHORITY (A3)

Receives:
    Artifact Candidate from A2

Performs:
    Sign Artifact
    Register in Artifact Registry
    Distribute Artifact
    Manage Artifact Versions


明确边界:

Build 输出 = Artifact Candidate
Artifact 输入 = Artifact Candidate
Artifact 输出 = Signed Artifact

A2 不知道 A3 的实现细节。
A3 不知道 A2 的实现细节。
```

### 6.1 绝对禁止的操作

```
FORBIDDEN:

  # Source Mutation
  modify source code
  commit to source repository
  merge to source branches

  # Protocol Mutation
  modify protocol definitions
  modify constitution
  modify contracts

  # Runtime Contract Mutation
  modify runtime contract
  modify AGENTS.md
  modify AGENT_CONTEXT.md

  # Artifact Mutation
  modify artifact after signing
  regenerate checksum after signing
  modify manifest after signing

  # Installation
  install runtime
  activate runtime instance
  configure runtime environment

  # Environment
  reference $env:AOS_HOME
  reference $HOME
  reference $USERPROFILE
  reference any user path
```

### 6.2 允许的操作

```
ALLOWED:

  read source (只读)
  read protocol (只读)
  compile source
  package artifact (.pkg)
  generate checksum (SHA256)
  generate artifact.manifest.json
  generate signature
  validate against protocol contract
  run tests
```

## 7. 版本模型

### 7.1 Factory 版本由 aos-runtime 决定

```
Build 不修改版本号。
Build 不自动升级版本号。
Build 只读取 Runtime Source 的 VERSION 文件。
Build 使用读取的版本号生成对应名称的 Artifact。

版本号格式: MAJOR.MINOR.PATCH
  MAJOR: Protocol 不兼容变更
  MINOR: 新功能，向后兼容
  PATCH: Bug 修复，向后兼容
```

### 7.2 版本写入 Manifest

Build 读取的版本号写入 artifact.manifest.json 的 `version_model`：

```
version_model.runtime_version = VERSION (from aos-runtime)
version_model.protocol_version = protocol version (from Protocol Artifact)
version_model.artifact_version = artifact version (generated by Build)
version_model.cli_version = cli version (from Runtime Source)
version_model.independent = true
```

## 8. 与现有 Contract 的关系

### 8.1 与 Artifact Contract 的关系

- Artifact Contract §2.3 定义 A2 Build Authority 的职责和边界
- Build Contract 是 A2 的具体执行契约
- Build 生成的 manifest 必须遵循 Artifact Contract §3 的 Schema
- Build 不签名（签名属于 A3），但生成 checksum 供 A3 使用

### 8.2 与 Runtime Contract 的关系

- Runtime Contract §2.1 CLI Interface 定义 `install` 命令 — Build 不触发 install
- Runtime Contract §3 版本模型 — Build 读取版本号，不修改版本号
- Runtime Contract §4 #1 "Runtime 不定义协议" — Build 同理，Build 不定义协议

### 8.3 与 Skill Contract 的关系

- Skill 的 `produced_by` 是 `aos-runtime` — Build 不修改 Skill 定义
- Skill 安装必须通过 Artifact — Build 只生产 Artifact，不安装 Skill

## 9. 不可违背约束

1. Build 不拥有 Source — Build 是 Source 的消费者
2. Build 不修改 Source — 只读访问
3. Build 不修改 Protocol — 只读访问
4. Build 不安装 Runtime — 只产生 Artifact Candidate
5. Build 不签名 Artifact — 签名属于 A3 Artifact Authority
6. Build 生成的 manifest 必须遵循 artifact-contract.md Schema
7. Build Pipeline 4 阶段不可跳过、不可合并
8. Build 不知道用户环境路径
9. Build 不知道 Installer 的存在
10. Build 版本号由 Source 决定，Build 不修改版本号