# AISE / Agent Governance 架构整理与开发规划（执行版）

> **版本**: v1.0.0-plan  
> **状态**: Planning — 待评审后执行  
> **日期**: 2026-07-15  
> **目标**: 把 `aise-standard` + `agent-governance` 整理为可冻结的 1.0 基线，不新增复杂概念，不提前扩展。

---

## 一、总体原则

本轮只做一件事：**整理现有架构，降低耦合，为 1.0 Frozen 打地基。**

坚持以下原则：

- 不增加新的复杂概念。
- 不提前设计未来数年的扩展能力。
- 优先整理现有架构。
- 优先降低耦合。
- 保证目录职责单一（Single Responsibility）。
- 保证后续能够稳定冻结版本。
- 保持与现有代码兼容，不进行破坏性重构。

所有优化必须围绕现有内容进行，禁止过度设计（Over Engineering）。

---

## 二、两个仓库职责边界（冻结目标）

### 2.1 aise-standard（标准仓库）

**定位**: «AI Agent Software Engineering Standard»

**职责**: 只描述 Agent 应该如何工作。

- 定义标准
- 定义规范
- 定义 Contract
- 定义 Policy
- 定义 Skill（触发词、流程、输入输出，不含实现）
- 定义 Template
- 定义 JSON Schema
- 定义版本与注册表

**原则**: 长期冻结资产（Frozen Standard）。

- ✅ 只放声明性内容（.md、.json、模板）
- ❌ 不放运行逻辑
- ❌ 不放安装逻辑
- ❌ 不放脚本实现
- ❌ 不负责治理
- ❌ 不负责项目修改
- ❌ 不感知 Governance
- ❌ 不感知具体项目

### 2.2 agent-governance（治理仓库）

**定位**: «Governance Runtime»

**职责**: 把 AISE Standard 应用于项目。

- Bootstrap（入口门）
- Install / Init（注入标准）
- Audit（审计）
- Update / Sync（同步标准）
- Migration（版本迁移）
- Validation（合规校验）
- Repository Governance（仓库身份、注册表、Handoff 索引）

**原则**: 运行时（Runtime）。

- ✅ 所有脚本、治理逻辑、安装逻辑、升级逻辑放在这里
- ✅ 引用 `aise-standard` 作为唯一标准源
- ❌ 不复制 Standard 内容
- ❌ 不存储项目级 Memory（应留在项目本地）
- ❌ 不替代 Standard 定义规则

### 2.3 项目关系

```text
Agent
  ↓
Agent Governance  ← Runtime：连接 Agent 与 Standard
  ↓
AISE Standard     ← Frozen：定义规则
  ↓
Project Repository ← 受治理项目
```

**关键约束**:

- AISE 不感知 Governance。
- AISE 不感知具体项目。
- Project 不感知 AISE 内部实现。
- Governance 是唯一负责连接三者的运行层。

---

## 三、当前目录结构盘点

### 3.1 aise-standard 当前结构

```text
aise-standard/
├── VERSION
├── README.md
├── SYSTEM.md                       # 系统提示词
├── Agent-Entry/
│   ├── Bootstrap.md                # Bootstrap Protocol
│   ├── COMPLIANCE.md
│   ├── AGENTS.md                   # Agent 入口声明
│   ├── CLAUDE.md
│   └── .trae/rules/aise.md         # IDE 规则
├── Contracts/                      # 4 合约
├── Policies/                       # 10 策略
├── Skills/                         # 8 Skill 定义
├── Registry/                       # 注册表
├── Git-Governance/
│   ├── git-gate.md                 # Git 关卡规则（标准）✅
│   ├── policies.json               # 关卡策略（标准）✅
│   ├── hooks/                      # Hook 脚本实现 ❌ 应移入 Governance
│   └── commands/                   # 命令脚本实现 ❌ 应移入 Governance
├── Migrations/
│   ├── v1.0.0_to_v1.1.0.md         # 迁移说明（标准）✅
│   └── migration.ps1               # 迁移脚本实现 ❌ 应移入 Governance
└── Templates/                      # 项目模板
```

### 3.2 agent-governance 当前结构

```text
agent-governance/
├── VERSION
├── README.md
├── PROJECT_BLUEPRINT.md
├── CHANGELOG.md
├── COMPATIBILITY.md
├── aise-template/                  # 项目初始化模板
│   ├── AISE/                       # ❌ 完整复制 aise-standard 内容
│   ├── AGENTS.md                   # ❌ 标准内容
│   ├── CLAUDE.md                   # ❌ 标准内容
│   ├── .trae/aise.md               # ❌ 标准内容
│   ├── PROJECT_BLUEPRINT.md
│   ├── CHANGELOG.md
│   ├── README.md
│   └── .gitignore
├── .agent-governance/              # 治理数据
│   ├── agent-contract.json         # Agent 生命周期规则 ✅
│   ├── agent-identity-contract.json # Agent 身份 ✅
│   ├── git-policy.json             # Git 策略 ✅
│   ├── repository-identities.json  # 仓库身份 ✅
│   ├── workspace-bindings.json     # 机器相关（不提交）✅
│   ├── workspace-registry.json     # 组合索引 ✅
│   ├── registry/projects.json      # 项目缓存 ✅
│   ├── handoff/                    # Handoff 历史/索引 ⚠️ 需明确为 Cache
│   ├── memory/                     # ❌ 项目 Memory 不应放在治理仓库
│   └── journal/                    # ⚠️ 需确认范围
├── scripts/                        # 治理脚本 ✅
│   ├── aise-init.ps1
│   ├── install-hooks.ps1
│   ├── agent_bootstrap.ps1
│   ├── agent_exit.ps1
│   ├── setup_agent_governance.ps1
│   └── audit_git_environment.ps1
├── schema/
│   └── handoff-schema.json         # ⚠️ Schema 建议移入 aise-standard
├── Templates/                      # .agent-entry.json 模板 ✅
├── tests/
│   └── smoke_test.ps1              # ✅
└── .github/
    └── workflows/
        └── governance-gate.yml     # ✅
    └── .github/workflows/          # ❌ 重复目录，应删除
```

---

## 四、问题标注与耦合分析

### 4.1 标准与运行时混放

| 位置 | 问题 | 原因 |
|------|------|------|
| `aise-standard/Git-Governance/hooks/` | Hook 脚本实现放在标准仓库 | Hook 是运行时执行逻辑，不属于声明性标准 |
| `aise-standard/Git-Governance/commands/` | `aise-verify.ps1` 等命令脚本放在标准仓库 | 脚本是 Governance Runtime 的实现 |
| `aise-standard/Migrations/migration.ps1` | 迁移脚本放在标准仓库 | 迁移执行属于 Governance Runtime |
| `aise-standard/Agent-Entry/.trae/rules/aise.md` | 存在争议：是标准内容还是 IDE 入口 | 建议保留在 Standard 作为入口模板，由 Governance 安装到项目 |

### 4.2 Standard 内容复制到 Governance

| 位置 | 问题 | 原因 |
|------|------|------|
| `agent-governance/aise-template/AISE/` | 完整复制 `aise-standard` 目录 | 双写导致标准更新后必须同步两份；违反单一事实源 |
| `agent-governance/aise-template/AGENTS.md` | 复制标准入口文件 | 应由 Governance 从 aise-standard 读取并生成 |
| `agent-governance/aise-template/CLAUDE.md` | 复制标准入口文件 | 同上 |
| `agent-governance/aise-template/.trae/aise.md` | 复制标准入口文件 | 同上 |

### 4.3 项目数据误放在 Governance

| 位置 | 问题 | 原因 |
|------|------|------|
| `agent-governance/.agent-governance/memory/` | 存放 `agent-workbench` 等项目的 Memory | 违反 `memory-ownership.md`：项目 Memory 必须属于项目本地 |
| `agent-governance/.agent-governance/journal/` | 存放 `architect.json` | 需确认是 Governance 自身日志还是项目日志；若是项目日志应迁移到项目本地 |

### 4.4 目录重复与冗余

| 位置 | 问题 | 原因 |
|------|------|------|
| `agent-governance/.github/.github/workflows/` | 目录嵌套重复 | 应为 `.github/workflows/` |
| `agent-governance/aise-template/` 与 `Templates/` | 职责重叠 | `aise-template/` 是完整项目骨架，`Templates/` 是 `.agent-entry.json` 模板；命名易混淆 |

### 4.5 标准仓库自身不完整

| 位置 | 问题 | 原因 |
|------|------|------|
| `aise-standard/PROJECT_BLUEPRINT.md` 缺失 | 标准仓库未遵循自身标准 | 每个 AISE 项目都应包含 PROJECT_BLUEPRINT.md |
| `aise-standard/.agent-entry.json` 缺失 | 标准仓库未声明自身治理入口 | 应声明自身由 agent-governance 治理 |

---

## 五、目标目录结构

### 5.1 aise-standard（标准仓库）目标结构

```text
aise-standard/
├── VERSION                         # 标准版本
├── README.md
├── PROJECT_BLUEPRINT.md            # 新增：标准仓库自身蓝图
├── .agent-entry.json               # 新增：指向 agent-governance
├── SYSTEM.md                       # 系统提示词
├── Agent-Entry/                    # Agent 入口模板
│   ├── Bootstrap.md                # Bootstrap Protocol（Frozen）
│   ├── COMPLIANCE.md
│   ├── AGENTS.md
│   ├── CLAUDE.md
│   └── .trae/rules/aise.md
├── Contracts/                      # 4 合约
├── Policies/                       # 10 策略
├── Skills/                         # 8 Skill 定义
├── Registry/                       # 注册表
├── Git-Governance/                 # 仅保留声明性规则
│   ├── git-gate.md                 # Git 关卡规则
│   └── policies.json               # 关卡策略
├── Migrations/                     # 仅保留迁移说明文档
│   └── v1.0.0_to_v1.1.0.md
├── Templates/                      # 项目模板
│   ├── PROJECT_BLUEPRINT.template.md
│   ├── CHANGELOG.template.md
│   ├── README.template.md
│   └── HANDOFF.template.md
└── Schema/                         # 新增或保留：JSON Schema
    └── handoff-schema.json
```

**移除项**:

- `Git-Governance/hooks/` → 移入 `agent-governance/scripts/hooks/`
- `Git-Governance/commands/` → 移入 `agent-governance/scripts/`
- `Migrations/migration.ps1` → 移入 `agent-governance/scripts/migrations/`

### 5.2 agent-governance（治理仓库）目标结构

```text
agent-governance/
├── VERSION                         # 治理运行时版本
├── README.md
├── PROJECT_BLUEPRINT.md
├── CHANGELOG.md
├── COMPATIBILITY.md
├── .agent-entry.json               # 指向自身治理
├── .agent-governance/              # 治理数据
│   ├── agent-contract.json
│   ├── agent-identity-contract.json
│   ├── git-policy.json
│   ├── repository-identities.json
│   ├── workspace-bindings.json     # 不提交
│   ├── workspace-registry.json
│   ├── registry/
│   │   └── projects.json           # Cache
│   ├── handoff/                    # Handoff 索引与历史（Cache）
│   │   ├── index.json
│   │   └── {project}/
│   │       ├── latest.json
│   │       └── history/
│   └── journal/                    # Governance 自身活动日志
│       └── 2026/
│           └── 07/
│               └── 15.json
├── scripts/                        # 治理脚本
│   ├── aise-init.ps1               # 从 aise-standard 注入标准
│   ├── install-hooks.ps1           # 安装 Git Hooks
│   ├── agent_bootstrap.ps1         # 入口门
│   ├── agent_exit.ps1              # 出口门
│   ├── setup_agent_governance.ps1  # 治理框架初始化
│   ├── audit_git_environment.ps1   # Git 环境审计
│   ├── hooks/                      # 从 aise-standard 移入
│   │   ├── pre-commit / pre-commit.ps1
│   │   ├── commit-msg / commit-msg.ps1
│   │   ├── pre-push / pre-push.ps1
│   │   ├── post-merge / post-merge.ps1
│   │   └── aise-audit.psm1
│   └── migrations/                 # 从 aise-standard 移入
│       └── v1.0.0_to_v1.1.0.ps1
├── templates/                      # 项目初始化骨架
│   ├── project-skeleton/           # 原 aise-template/ 改名
│   │   ├── PROJECT_BLUEPRINT.md
│   │   ├── CHANGELOG.md
│   │   ├── README.md
│   │   └── .gitignore
│   └── agent-entry/
│       └── .agent-entry.template.json
├── tests/
│   └── smoke_test.ps1
└── .github/
    └── workflows/
        └── governance-gate.yml
```

**移除项**:

- `aise-template/AISE/` → 改为运行时从 `aise-standard` 读取
- `aise-template/AGENTS.md`、`CLAUDE.md`、`.trae/aise.md` → 改为运行时从 `aise-standard/Agent-Entry/` 复制
- `.agent-governance/memory/` → 迁移到各项目本地 `.project/memory/`
- `.agent-governance/journal/architect.json` → 如为项目日志，迁移到项目本地 `.project/journal/`
- `.github/.github/` → 删除重复目录
- `schema/` → 移入 `aise-standard/Schema/`（如 Schema 属于标准定义）

---

## 六、解耦方案

### 6.1 标准与运行时解耦

**原则**: Standard 只声明规则，Governance 负责实现。

| 当前耦合点 | 解耦方式 |
|-----------|---------|
| `aise-standard/Git-Governance/hooks/` | 迁移到 `agent-governance/scripts/hooks/` |
| `aise-standard/Git-Governance/commands/` | 迁移到 `agent-governance/scripts/` |
| `aise-standard/Migrations/migration.ps1` | 迁移到 `agent-governance/scripts/migrations/` |
| `agent-governance/aise-template/AISE/` | 删除复制；`aise-init.ps1` 改为从 `aise-standard` 仓库读取 |
| `agent-governance/schema/handoff-schema.json` | 迁移到 `aise-standard/Schema/handoff-schema.json` |

### 6.2 项目数据与治理数据解耦

**原则**: 项目 Memory / Journal / Handoff Primary 属于项目本地，Governance 只保留索引/Cache。

| 当前耦合点 | 解耦方式 |
|-----------|---------|
| `agent-governance/.agent-governance/memory/` | 迁移到各项目 `.project/memory/`；Governance 保留只读索引或删除 |
| `agent-governance/.agent-governance/journal/architect.json` | 如为项目活动日志，迁移到项目 `.project/journal/` |
| `agent-governance/.agent-governance/handoff/` | 明确为 Cache/Index；Primary 保留在项目本地 `.project/handoff/` 或 `.project/context/handoff/` |

### 6.3 版本依赖解耦

**原则**: Governance 引用 Standard 版本，不绑定固定路径。

- `agent-governance/VERSION` 独立演进。
- `agent-governance/COMPATIBILITY.md` 声明支持的 `aise-standard` 版本范围。
- `aise-init.ps1` 通过 `aise-standard/VERSION` 和 `COMPATIBILITY.md` 决定可注入版本。
- 禁止 `aise-template` 内置固定版本 AISE。

### 6.4 命名解耦

| 当前名称 | 建议名称 | 原因 |
|---------|---------|------|
| `agent-governance/aise-template/` | `agent-governance/templates/project-skeleton/` | 避免与 `aise-standard` 名称混淆，明确是项目骨架 |
| `agent-governance/Templates/` | `agent-governance/templates/agent-entry/` | 与 project-skeleton 合并到 templates/ 下，职责清晰 |

---

## 七、推荐的 Git 分支治理方案（Main + Develop）

### 7.1 分支模型

每仓库只维护两条主线：

```text
main
  ↑
develop  ← 所有日常开发
  ↑
feature/*
fix/*
refactor/*
```

### 7.2 main 分支

- 冻结版本。
- 所有发布 Tag 从 main 创建。
- 禁止直接提交。
- 只能通过 `--no-ff` merge 从 develop 进入。

### 7.3 develop 分支

- 所有开发工作默认分支。
- feature / fix / refactor 分支从此拉出。
- 验证完成后合并回 develop。
- 达到冻结条件后，merge 到 main 并打 Tag。

### 7.4 临时分支

- `feature/*`：新功能
- `fix/*`：Bug 修复
- `refactor/*`：重构
- 合并后删除。

### 7.5 当前阶段不引入

- 不引入 `release/*` 分支。
- 不引入 `preview`、`experimental` 分支。
- 不引入复杂 Git Flow。

### 7.6 提交与标签规则

- Commit message: `<type>(<scope>): <summary> [hint:<keyword>]`
- Tag: `v{MAJOR}.{MINOR}.{PATCH}`
- 仅推送当前分支与当前 Tag，禁止 `--tags`。

---

## 八、未来产品方向（仅设计目标，当前不实现）

最终形成两种部署方式，但基于同一套 AISE Standard + Governance Runtime：

### 8.1 Standalone Runtime（单 Agent）

- 直接接入 Claude、GPT、Gemini、Trae、Cursor、Codex 等单实例 Agent。
- Governance 以本地脚本形式运行。
- 适合个人开发者、单项目。

### 8.2 Gateway Runtime（多 Agent）

- 通过统一 Gateway 管理多个 Agent。
- Governance 以服务形式运行。
- 适合团队、多项目、远程 Agent。

### 8.3 设计约束

- Standard 保持统一。
- Governance 保持统一。
- Runtime 根据部署方式适配不同入口。
- 不复制代码。
- 不维护双版本。

当前阶段只保证架构不阻塞未来两种模式，不实现 Gateway Runtime。

---

## 九、本轮执行步骤（非破坏性）

### Phase 1: 文档与元数据补齐

1. 在 `aise-standard` 新增 `PROJECT_BLUEPRINT.md`。
2. 在 `aise-standard` 新增 `.agent-entry.json`，指向 `agent-governance`。
3. 更新 `aise-standard/README.md` 中的目录结构描述。
4. 更新 `agent-governance/PROJECT_BLUEPRINT.md` 中的目录结构描述。

### Phase 2: 标准仓库清理（aise-standard）

1. 将 `aise-standard/Git-Governance/hooks/` 移入 `agent-governance/scripts/hooks/`。
2. 将 `aise-standard/Git-Governance/commands/` 移入 `agent-governance/scripts/`。
3. 将 `aise-standard/Migrations/migration.ps1` 移入 `agent-governance/scripts/migrations/`。
4. `aise-standard/Git-Governance/` 仅保留 `git-gate.md` + `policies.json`。
5. 将 `agent-governance/schema/handoff-schema.json` 移入 `aise-standard/Schema/handoff-schema.json`。

### Phase 3: 治理仓库清理（agent-governance）

1. 重命名 `agent-governance/aise-template/` → `agent-governance/templates/project-skeleton/`。
2. 重命名 `agent-governance/Templates/` → `agent-governance/templates/agent-entry/`。
3. 删除 `templates/project-skeleton/AISE/` 复制内容。
4. 删除 `templates/project-skeleton/AGENTS.md`、`CLAUDE.md`、`.trae/aise.md`。
5. 删除 `.github/.github/` 重复目录。
6. 将 `.agent-governance/memory/` 迁移到各项目本地 `.project/memory/`。
7. 明确 `.agent-governance/handoff/` 为 Cache，Primary 在项目本地。
8. 如 `.agent-governance/journal/architect.json` 为项目日志，迁移到项目本地。

### Phase 4: 脚本改造

1. 改造 `aise-init.ps1`：
   - 不再从 `aise-template/AISE/` 复制。
   - 从 `aise-standard` 仓库读取 `Agent-Entry/` 和 `AISE/` 内容并注入。
   - 支持版本锁定（读取 `AISE/VERSION`）。
2. 改造 `install-hooks.ps1`：
   - 从 `agent-governance/scripts/hooks/` 安装 hooks。
3. 改造 `agent_bootstrap.ps1` / `agent_exit.ps1`：
   - Memory / Journal / Handoff Primary 读写路径改为项目本地。

### Phase 5: 验证

1. 在测试项目运行 `aise-init.ps1`，验证注入内容完整。
2. 运行 `smoke_test.ps1`，验证 Governance Runtime 基本功能。
3. 运行 `aise-verify.ps1`，验证标准结构完整。
4. 检查无 `aise-template/AISE/` 复制内容。
5. 检查 `.agent-governance/memory/` 已清空或仅保留索引。

### Phase 6: 存档与冻结

1. 分别对 `aise-standard` 和 `agent-governance` 执行 Archive。
2. 各自打 Tag `v1.0.0`。
3. 更新 `COMPATIBILITY.md` 声明 `aise-standard v1.0.0` 与 `agent-governance v1.0.0` 兼容。
4. 进入 1.0 Frozen 状态。

---

## 十、兼容性说明

- 所有现有项目（已注入 AISE）的结构不变，无需回滚。
- `aise-init.ps1` 改造后，新注入项目的行为保持一致（仍生成 `AISE/` 目录和入口文件）。
- `agent-governance/scripts/` 路径保持兼容，只是脚本内部实现从读取本地模板改为读取 `aise-standard`。
- 旧的 `aise-template/AISE/` 内容删除后，若历史项目需要修复，可通过 `aise sync` 从 `aise-standard` 重新同步。
- 不进行破坏性重构：不删除 `.git/hooks`，不修改已提交历史，不强制重置项目。

---

## 十一、风险与注意事项

| 风险 | 缓解措施 |
|------|---------|
| `aise-init.ps1` 改造后路径变化导致失败 | 先在测试项目验证，再推广 |
| 删除 `aise-template/AISE/` 后离线不可用 | Governance 本地缓存可选保留 `aise-standard` 最新副本 |
| 项目 Memory 迁移可能丢失 | 迁移前备份，迁移后验证 |
| Hook 脚本移动后路径引用错误 | 全局搜索并替换路径引用 |
| aise-standard 自身无蓝图 | Phase 1 立即补齐 |

---

## 十二、总结

本轮目标：**把地基打牢**。

核心动作：

1. **职责拆分**: `aise-standard` 只保留声明性标准，`agent-governance` 只保留运行时实现。
2. **消除复制**: `aise-template/AISE/` 不再复制标准内容，改为运行时引用 `aise-standard`。
3. **项目数据归位**: Memory / Journal / Handoff Primary 回到项目本地，Governance 只保留索引。
4. **目录清理**: 删除重复目录、重命名易混淆目录、补齐标准仓库自身蓝图。
5. **分支治理**: 采用 `main` + `develop` 最简模型，为 1.0 Frozen 做准备。

完成后，两个仓库进入 1.0 Frozen 候选状态，后续再基于稳定基线派生 Standalone Runtime 与 Gateway Runtime。
