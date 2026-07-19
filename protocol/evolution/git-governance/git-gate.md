# AISE Git Governance Layer

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

Git Governance Layer 是 AISE 的 Git 原生出入口控制层。它从"AI 遵守规则"升级到"系统保证规则"，确保 Git 成为最终资产边界（Asset Boundary）。

## 2. 七道关卡架构

```
              Agent
                |
        AISE Bootstrap Gate
                |
             Workspace
                |
    ┌───────────┼───────────┐
    │           │           │
 Pull Gate   Commit      Push Gate
(commands)    Gate      (pre-push)
    │           │           │
aise-fetch   pre-commit   Remote
aise-update  commit-msg   GitHub
aise-sync        │
    │           ↓
    │       Local Git
    │
    ↓
Post-Merge Gate
(post-merge)

  所有关卡之上：
  Authorization Gate (豁免层)
  Audit Gate (审计层)
```

## 3. 关卡 1：Pull / Fetch Gate（入口 — Commands 层）

**注意**：Git 原生没有 `pre-pull` / `pre-fetch` hook。此关卡通过 AISE Commands 替代直接 `git pull`。

| 命令 | 替代 | 说明 |
|------|------|------|
| `aise fetch` | `git fetch` | 获取远程变更，工作区脏时警告但允许继续 |
| `aise update` | `git pull` | 工作区脏时阻止，必须干净才能更新 |
| `aise sync` | 完整同步流程 | Y0-Y7 完整 Sync Protocol，含备份和验收 |

## 4. 关卡 2：Commit Gate（pre-commit）

触发时机：`git commit` 之前

| 检查 | 说明 | 豁免 |
|------|------|------|
| 敏感文件 | 禁止 `.env`、`secrets/`、`*.key`、`*.pem` | P0 不可豁免 |
| CHANGELOG 同步 | 代码变更时要求 CHANGELOG.md 同步更新 | P1 可豁免 |
| 测试检查 | 可选，默认关闭 | — |

## 5. 关卡 3：Commit Message Gate（commit-msg）

格式强制：`<type>(<scope>): <summary>`

允许类型：`feat`、`fix`、`chore`、`docs`、`refactor`、`test`、`infra`、`perf`

## 6. 关卡 4：Push Gate（pre-push）

| 检查 | 说明 | 豁免 |
|------|------|------|
| Remote 验证 | 必须为 `origin`（GitHub） | P1 可豁免 |
| 禁止 Force Push | 禁止 `--force`、`-f`、`--force-with-lease` | P1 可豁免 |
| 禁止 `--tags` | 禁止推送全部标签 | P0 不可豁免 |
| 保护分支 | 禁止删除 `main`/`master` | P0 不可豁免 |
| Archive 状态 | 按分支区分：main/release 警告，feature 跳过 | — |

### Archive 检查分支策略

| 分支 | 规则 |
|------|------|
| `main` / `master` | 警告：HEAD 未打 tag 时提示 `archive` |
| `release/*` | 警告：HEAD 未打 tag 时提示 `archive` |
| `feature/*` / `feat/*` / `fix/*` | 跳过：允许直接 push |

## 7. 关卡 5：Merge Gate（post-merge）

| 检查 | 说明 |
|------|------|
| AISE 版本 | 检查合入后版本是否变化 |
| 依赖检查 | 检查是否有新依赖 |
| 测试状态 | 提示是否需要重新运行测试 |
| Merge Report | 生成合并报告 |

## 8. 关卡 6：Authorization Gate（豁免层）

**权限等级**：

| 级别 | 含义 | 示例 |
|------|------|------|
| **P0** | 安全底线 | 泄露 secret、删除保护分支 |
| **P1** | 工程操作 | 修改 remote、force push |
| **P2** | 自动允许 | 更新 docs、调整代码 |

**豁免流程**：详见 `Policies/exemption-policy.md`

## 9. 关卡 7：Audit Gate（审计层）

所有治理事件写入 `.project/audit/`：

```
.project/audit/
├── gate-events.jsonl          # Hook 执行记录
├── agent-events.jsonl         # Agent 行为记录
└── authorization-events.jsonl # 豁免授权记录
```

## 10. 三层部署架构

```
第一层：agent-governance 远程标准库
├── aise-standard/Git-Governance/     ← 标准模板（法律）
│   ├── git-gate.md
│   ├── policies.json
│   ├── hooks/（shell wrapper + .ps1 + aise-audit.psm1）
│   └── commands/（fetch/update/sync/verify）

第二层：项目 AISE 快照
├── AISE/Git-Governance/              ← 项目绑定版本

第三层：Git 实际 Hook
├── .git/hooks/                       ← 真正执行拦截
```

## 11. 版本锁定

- Git Governance 版本随 AISE 版本锁定
- 升级由项目决定，不自动漂移
- 迁移脚本在 `AISE/Migrations/` 中管理

## 12. 变更控制

进入 Frozen 后：
- 允许新增关卡
- 允许新增检查项
- 禁止删除已有关卡
- 禁止放宽 P0 检查项