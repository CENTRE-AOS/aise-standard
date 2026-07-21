# Production Repository Bootstrap Contract

> Version: 2.0.0-frozen
> Date: 2026-07-21
> Phase: S0.8 — Bootstrap Contract Freeze (Revised)
> Status: FROZEN

---

## 1. 目的

定义四个生产 Authority 仓库的最小文件集。Bootstrap Contract 分为两层：
- **Common Identity Layer** — 所有生产仓库必须
- **Authority Bootstrap Layer** — 按 Authority 类型不同

---

## 2. Common Identity Layer（所有生产仓库必须）

| 文件 | 用途 |
|------|------|
| `VERSION` | 当前版本号 |
| `README.md` | 仓库说明 |
| `CHANGELOG.md` | 变更日志 |
| `.gitignore` | Git 忽略规则 |
| `BOOTSTRAP.md` | 引导协议 |
| `PROJECT_BLUEPRINT.md` | 架构蓝图 |
| `.github/remote-identity.json` | 仓库身份声明 |
| `.handoff/` | 交接上下文 |

---

## 3. Authority Bootstrap Layer（按 Authority 类型）

### 3.1 A0 Protocol Authority (aise-standard)

**角色**: 治理标准定义者。Agent 可在此仓库执行 Governance 操作。

| 文件/目录 | 用途 |
|-----------|------|
| `PROJECT_CONTEXT.md` | 项目上下文 |
| `PROJECT_STATE.json` | 项目状态 |
| `AGENTS.md` | Agent 行为规则 |
| `AGENT_CONTEXT.md` | Agent 身份契约 |
| `.agent-entry.json` | AISE Protocol Manifest |
| `.project/centre.protocol.json` | CENTRE Protocol Manifest |
| `constitution/` | 宪法与 Contract |
| `protocol/` | RFC 与 Protocol 定义 |
| `registry/` | Protocol Registry |
| `templates/` | 模板文件 |

### 3.2 A1 Source Authority (aos-runtime)

**角色**: Runtime 源代码提供者。Agent 可在此仓库执行 Governance 操作。

| 文件/目录 | 用途 |
|-----------|------|
| `PROJECT_CONTEXT.md` | 项目上下文 |
| `PROJECT_STATE.json` | 项目状态 |
| `AGENTS.md` | Agent 行为规则 |
| `AGENT_CONTEXT.md` | Agent 身份契约 |
| `.agent-entry.json` | AISE Protocol Manifest |
| `runtime.manifest.json` | Runtime Manifest |
| `runtime/` | Runtime 核心代码 |
| `Skills/` | Skill 实现 |
| `tests/` | 测试代码 |

### 3.3 A2 Build Authority (aos-factory)

**角色**: 生产构建器。**不是 Agent Runtime Node**。Agent 不在此仓库执行 Governance，仅执行 Build Pipeline。

| 文件/目录 | 用途 |
|-----------|------|
| `factory.manifest.json` | Factory Manifest |
| `artifact.schema.json` | Artifact Schema |
| `builder/` | 构建引擎 |
| `pipeline/` | CI/CD Pipeline 定义 |
| `validator/` | 验证器 |

**不需要**: AGENTS.md / AGENT_CONTEXT.md / PROJECT_CONTEXT.md / PROJECT_STATE.json / .agent-entry.json — Factory 不是 Agent Runtime Node。

### 3.4 A4 Install Authority (aos-installer)

**角色**: 部署工具。**不是 Agent Runtime Node**。Agent 不在此仓库执行 Governance，仅执行 Install Pipeline。

| 文件/目录 | 用途 |
|-----------|------|
| `installer.manifest.json` | Installer Manifest |
| `cli/` | CLI 工具 |
| `engine/` | 安装引擎 |
| `manifest/` | Installer 配置 |
| `registry/` | Installer Registry |

**不需要**: AGENTS.md / AGENT_CONTEXT.md / PROJECT_CONTEXT.md / PROJECT_STATE.json / .agent-entry.json — Installer 不是 Agent Runtime Node。

---

## 4. 当前状态与清理建议

| 仓库 | 多余文件（迁移残留） | 处理 |
|------|---------------------|------|
| aos-factory-new | AGENTS.md, AGENT_CONTEXT.md, PROJECT_CONTEXT.md | 不阻塞 S1。S2 后评估是否移除。Factory 不需要充当 Agent Node |
| aos-installer | AGENTS.md, AGENT_CONTEXT.md, PROJECT_CONTEXT.md | 不阻塞 S1。S2 后评估是否移除。Installer 不需要充当 Agent Node |

**约束**: 这些文件不阻塞 S1 创建 GitHub。在 S2 Remote Binding 后评估清理。不在此 Phase 执行删除。

---

## 5. S1 创建 GitHub 后的即时检查表

```
□ VERSION
□ README.md
□ CHANGELOG.md
□ .gitignore
□ BOOTSTRAP.md
□ PROJECT_BLUEPRINT.md
□ .github/remote-identity.json
```

四项检查全部 PASS。

---

## 6. 不可违背约束

1. Factory 和 Installer 不是 Agent Runtime Node — Bootstrap Contract 不要求 AGENTS/AGENT_CONTEXT/PROJECT_STATE
2. Protocol 和 Runtime 是 Agent Governance Node — 必须包含完整 Agent 上下文
3. Archive (aos-migration) 不受此 Contract 约束
4. 文件清理在 S2 后评估，不阻塞 S1