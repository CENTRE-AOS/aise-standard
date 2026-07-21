# CENTRE Installer Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-21
> 适用范围: CENTRE Foundation v3.2.0
> 前置: Artifact Contract v1.0.0, Build Contract v1.0.0
> 关联: install-authority-map.json, artifact-authority-map.json

---

## 1. 定义

CENTRE Installer Contract 定义 A4 Install Authority 的输入边界、验证契约、安装流程和禁止权力。Installer 是 Artifact → Runtime Instance 的唯一入口，由 aos-installer 执行。

**Installer 是 Artifact 的消费者，不是 Source 的消费者。**

```
Artifact Authority (A3)
        │
        │ provides Signed Artifact
        ▼
Install Authority (A4)
        │
        │ creates Runtime Instance
        ▼
    Runtime Instance
```

## 2. Install Identity

| 属性 | 值 |
|------|-----|
| Owner | aos-installer |
| Consumer of | A3 Artifact Authority |
| Authority | INSTALL_AUTHORITY |
| 职责 | 验证 Artifact 完整性、安装到 Runtime Instance、激活 |
| 输入 | Signed Artifact (artifact_state = signed) |
| 产出 | Runtime Instance (artifact_state = installed) |
| 禁止 | git clone source、npm install source、pip install source、直接修改 Runtime |

### 2.1 Install Authority 不是什么

| 不是 | 原因 |
|------|------|
| 不是 Build System | Build 属于 A2 (aos-factory-new) |
| 不是 Artifact Signer | Sign 属于 A3 (Artifact Authority) |
| 不是 Runtime | 不执行 Agent 生命周期、Skill 执行 |
| 不是 Agent Manager | Agent 生命周期由 Runtime 管理 |
| 不是 Protocol Authority | Protocol 属于 aise-standard (A0) |
| 不是 Source Consumer | Source 属于 A1，Installer 不直接消费 Source |

## 3. Install Input Contract

### 3.1 唯一合法输入

```
Installer 唯一输入:

Signed Artifact (.pkg)
  - 来源: A3 Artifact Authority
  - 格式: aos-runtime-{version}.pkg
  - 必须包含: artifact.manifest.json (遵循 artifact-contract.md §3)
  - 必须满足: artifact_state = signed
```

### 3.2 禁止的输入

```
FORBIDDEN:

  Source Repository (git clone)
  Git URL
  Package Registry Source Code
  Build Output Directory
  Artifact Candidate (artifact_state = candidate)
  Unverified Source Archive
  npm/pip/curl/wget source
  Direct File Copy
```

### 3.3 输入验证流程

```
1. 验证 artifact.manifest.json 存在且有效
2. 验证 artifact_state == "signed"
3. 验证 checksum 匹配 (SHA256)
4. 验证 signature 有效 (rsa-sha256)
5. 验证 version_model.independent == true
6. 验证 compatibility 满足
7. 验证 bootstrap_contract 满足
8. 验证 immutable == true

任何步骤失败 → 终止安装
```

## 4. Install Output Contract

### 4.1 允许的输出

```
Installer 输出:

Output #1: Runtime Instance
  - 位置: AOS_HOME (由 Installer 创建)
  - 内容: runtime/* + Skills/* + system-skills/*
  - 状态: artifact_state = installed

Output #2: Instance Identity
  - 格式: device.cert + instance.cert + instance.key
  - 生成时机: 安装时动态生成

Output #3: Install Report
  - 格式: install-report.json
  - 内容: timestamp, artifact info, validation results, installed files
```

### 4.2 禁止的输出

```
FORBIDDEN:

  Modified Artifact
  Modified Manifest
  Modified Runtime Code
  Bundled Certificates (证书在安装时动态生成)
  Environment-specific Configuration
```

## 5. Install Pipeline Contract

### 5.1 Pipeline 阶段（冻结）

```
Stage 1: Fetch
  从 A3 Artifact Authority 获取 Signed Artifact

Stage 2: Validate
  验证 artifact.manifest.json 完整性
  验证 artifact_state == "signed"
  验证 checksum (SHA256)
  验证 signature (rsa-sha256)
  验证 version_model 独立
  验证 compatibility 满足
  验证 bootstrap_contract 满足

Stage 3: State Transition → verified
  验证通过后，artifact_state: signed → verified

Stage 4: Install
  解压 Artifact 到 AOS_HOME
  创建目录结构
  部署文件

Stage 5: State Transition → installed
  安装完成后，artifact_state: verified → installed

Stage 6: Identity Setup
  生成 Instance Identity (device.cert + instance.cert + instance.key)

Stage 7: Activate
  激活 Runtime Instance
```

### 5.2 Pipeline 约束

- 7 个阶段不可跳过、不可合并
- 任何阶段失败 → 终止 Pipeline
- Install 不修改 Artifact 内容
- Install 不修改 Manifest
- Install 不自我判断 Identity/Permission/Policy（委托 CENTRE Kernel）
- State Transition 必须通过 Authority Action，不可直接修改 artifact_state 字段

## 6. Install Authority Forbidden Scope

### 6.1 绝对禁止的操作

```
FORBIDDEN:

  # 绕过 Artifact
  install from git clone
  install from npm/pip/curl/wget
  install from source directory
  install from build output

  # 跳过验证
  skip checksum verification
  skip signature verification
  skip artifact_state check
  skip compatibility check

  # 直接修改状态
  modify artifact_state without Authority Action
  directly set artifact_state = "installed"

  # 修改 Artifact
  modify artifact.manifest.json
  modify runtime/kernel/*
  modify protocol/*
  modify constitution/*

  # 修改 Runtime
  directly modify installed runtime
  patch installed runtime
  rebuild installed runtime

  # 自我判断
  self-judge Identity validity
  self-judge Permission validity
  self-judge Policy compliance
```

### 6.2 允许的操作

```
ALLOWED:

  fetch signed artifact from A3
  validate artifact integrity (checksum + signature)
  transition artifact_state (通过 Authority Action)
  extract artifact to AOS_HOME
  create directory structure
  deploy files
  generate instance identity
  activate runtime instance
  delegate admission to CENTRE Kernel
```

## 7. State Transition Contract

### 7.1 Installer 负责的状态转换

```
signed → verified     (A4: Validate 通过后)
verified → installed  (A4: Install 完成后)
```

### 7.2 Installer 禁止的状态操作

```
FORBIDDEN:

  candidate → signed    (A3 的职责)
  candidate → verified  (跳过签名)
  candidate → installed (跳过签名和验证)
  signed → installed    (跳过验证)
  installed → *         (不可逆)
  directly modify artifact_state field
```

### 7.3 状态转换规则

```
State Transition MUST be caused by Authority Action.
State Mutation IS FORBIDDEN.

State 是 Authority Action 的产物，不是可独立修改的字段。
```

## 8. 与现有 Contract 的关系

### 8.1 与 Artifact Contract 的关系

- Artifact Contract §2.5 定义 A4 Install Authority 的职责和边界
- Artifact Contract §3.3 定义 artifact_state 生命周期状态
- Artifact Contract §3.4 定义 State Mutation Contract
- Installer 必须遵循 artifact-contract.md 的所有状态转换规则

### 8.2 与 Build Contract 的关系

- Build Contract §2 定义 A2 Build Authority — Installer 不执行 Build
- Build 产生 Artifact Candidate — Installer 只接受 Signed Artifact
- A2 和 A4 之间通过 A3 连接，无直接交互

### 8.3 与 Runtime Contract 的关系

- Runtime Contract §2.1 CLI Interface 定义 `install` 命令 — Installer 实现该命令
- Runtime Contract §2.4 Registry Interface — Installer 管理 Bootstrap Discovery Registry
- Runtime Contract §3 版本模型 — Installer 验证但不修改

## 9. 两个注册表（不可混淆）

| 注册表 | 所属 | 生命周期 | 职责 |
|--------|------|---------|------|
| Bootstrap Discovery Registry | Installer | 安装阶段 | 发现 Agent 环境位置 |
| Runtime Adapter Registry | Runtime | 运行阶段 | 与 Agent 环境交互 |

## 10. 不可违背约束

1. Install 只接受 Signed Artifact (artifact_state = signed) — 不接受 Source
2. Install 不绕过验证 — checksum + signature + compatibility 必须全部通过
3. Install 不修改 Artifact 内容 — 只读
4. Install 不修改 Manifest — 只读
5. Install 不自我判断 Identity/Permission/Policy — 委托 CENTRE Kernel
6. State Transition 必须通过 Authority Action — 不可直接修改 artifact_state
7. Install 不修改 Runtime Code — 只部署
8. Install 不生成证书 — 证书在安装时由 Runtime 动态生成
9. Install 不假设 Agent 环境路径 — 不硬编码路径
10. Install 不绑定单一 Agent Vendor — 支持多 Agent 平台