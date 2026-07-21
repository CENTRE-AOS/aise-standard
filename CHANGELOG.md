# Changelog

All notable changes to aise-standard will be documented in this file.

## [2.0.0-frozen] — 2026-07-20 — P12-C0 Bootstrap Restore Protocol

### feat: runtime-contract Bootstrap 生命周期

- `constitution/constitution/contracts/runtime-contract.md`: 新增 Bootstrap 生命周期（BIRTH / RESTORE / RECOVERY）
  - 定义 Bootstrap Engine 三态模型
  - 约束：只有一个 Bootstrap Engine 实现（aise-bootstrap），不允许 handoff 自行实现 Bootstrap Restore
  - Handoff RESUME 必须调用 `aise-bootstrap.restore()`
  - Bootstrap Restore 是 Admission 的前置条件

## [2.0.0-frozen] — 2026-07-20 — P12-0.6 Capability Integration Freeze

### Registry 版本号修正

- `registry/skills.json`: aise_version 2.5.0frozen → 2.0.0-frozen，新增 4 个 CENTRE Skill（admission, freeze, health, update），新增 governance.provider
- `registry/version.json`: aise_standard 2.5.0 → 2.0.0-frozen，aise_protocol 1.0 → 2.0.0-frozen，governance.provider agent-governance → aos-runtime

### Constitution 引用修正

- `constitution/constitution/CONSTITUTION.md`: 适用范围 v3.1.0 → v3.2.0，aos-protocol-factory → aise-standard（全部引用修正）
- `constitution/constitution/contracts/runtime-contract.md`: Runtime 2.1.0 → 3.2.0，新增 2.6 Bootstrap Interface + 2.7 Governance Loop Interface
- `constitution/constitution/contracts/skill-contract.md`: 新增 IBootstrap 接口，新增 bootstrap:validate 权限，新增 Bootstrap Context Check

### Entry 更新

- `README.md`: 新增 Bootstrap Contract、Context Authority Model、Agent Understanding Layer 说明
- `.project/centre.protocol.json`: runtime_version v3.1.0 → v3.2.0

### 新资产

- `PROJECT_BLUEPRINT.md`: Protocol Authority 架构蓝图
- `PROJECT_STATE.json`: Protocol Authority 当前状态
- `.project/context/handoff/`: Handoff 目录 + Protocol Update Handoff

## [2.0.0-frozen] — 2026-07-20 — CENTRE Protocol Identity

### Protocol Identity Migration (P12-0.5)

- `.agent-entry.json`: Protocol Manifest（首次创建）
- `PROTOCOL_IDENTITY.md`: AISE 1.x → CENTRE Protocol 2.0.0 身份迁移声明
- `AGENTS.md`: Protocol Authority 行为规则（禁止修改 Frozen Contract）
- `AGENT_CONTEXT.md`: Repository Identity Contract（五级 Authority Ranking）
- 明确区分：Protocol Version (2.0.0-frozen) ≠ Constitution Version (1.0.0frozen)

### Context Authority Model (RFC-0010)

- `protocol/governance/context-authority-model.md`: 五级权威体系
- Layer 0: Protocol Authority (NEVER modify)
- Layer 1: Repository Identity
- Layer 2: Architecture Truth (RFC required)
- Layer 3: Implementation (Allowed)
- Layer 4: Historical Reference (Read-only)
- 双层 Bootstrap 模型: AGENTS.md → AGENT_CONTEXT.md

### Identity Fix (Step 7.5)

- `aos-protocol-factory` → `aise-standard` 身份修正
- `.project/centre.protocol.json` 更新
- 10 system-skills manifest.json 更新

---

## [2.0.0-frozen] — 2026-07-19

### Initial Release (Repository Extraction)

- Repository extracted from legacy monorepo (`legacy-monorepo-freeze-v3.1.0`)
- AOS Repository Separation Architecture v1.1.0 FROZEN
- Five-role isolation model established
- Three core boundaries: Authority, Artifact, Lifecycle
- Bootstrap Contract ratified (three-layer: Protocol defines, Runtime executes, Factory validates)
- Installer Bootstrap Layer concept introduced (Bootstrap Discovery Registry + Runtime Adapter Registry)
- 22 ADRs documented
- Protocol Registry established (compliance, routing, skills, version)
- 14 System Governance Skills defined
- Project Bootstrap Templates provided

### Migration Baseline

- Source: `legacy-monorepo-freeze-v3.1.0` (commit `ffded4c`)
- Method: git filter-repo extraction
- Preserved: 30 commits of git history