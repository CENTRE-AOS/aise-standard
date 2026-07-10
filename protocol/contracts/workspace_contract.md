# Workspace Contract

> 版本：v1.0
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

本合约定义 Workspace 的类型、状态、升级流程与数据保护规则，确保 Repository 版本能够安全部署为可运行状态。

## 2. Workspace 类型

| 类型 | 说明 | 数据保留 |
|------|------|---------|
| Local | 用户本地机器 | 长期保留 |
| Cloud | 临时 Cloud Agent 环境 | 不保留，销毁前必须 Commit + Push + HANDOFF |

## 3. Workspace State

Sync Workspace 维护 `.sync/` 目录：

```text
.sync/
    workspace_state.json      # 当前环境状态
    sync_history.json         # 同步历史
    artifact_manifest.json    # 产物清单
```

### workspace_state.json

```json
{
    "workspace": "Local",
    "branch": "main",
    "current_version": "v1.2.3",
    "last_sync": "2026-07-10T08:31:00+08:00",
    "git_commit": "abcd1234",
    "git_tag": "v1.2.3",
    "artifact_version": "v1.2.3",
    "artifact_path": "dist/app.exe",
    "database_version": 8,
    "config_version": 3
}
```

## 4. 升级流程

```text
Environment Identification
    ↓
Repository Sync
    ↓
Analyze
    ↓
Upgrade Plan
    ↓
Workspace Backup
    ↓
Upgrade
    ↓
Launch
    ↓
Acceptance
```

## 5. 数据保护

Workspace 升级时，以下数据必须完整保留：

- `config/` 目录
- `database/` 目录
- `feedback/` 目录
- 用户生成的任何非版本控制文件

升级前必须备份到 `.backup/YYYY-MM-DD-NNN/`。

## 6. Acceptance（Smoke Test）

Sync Workspace 完成后必须执行 Smoke Test，验证：

- Launch 成功
- 欢迎页面 / 主界面正常
- 核心功能可访问
- 状态栏 / 健康检查正常

## 7. 变更控制

本合约进入 Frozen 状态后：

- 允许新增 Workspace 类型或状态字段。
- 禁止修改升级流程顺序或数据保护规则。
