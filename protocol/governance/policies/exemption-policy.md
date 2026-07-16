# AISE Exemption Protocol

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

AISE 治理体系是硬防线，但完整的工程治理需要**最高授权通道**。当用户 Mission 与 Policy 冲突时，Exemption Protocol 提供正式、可审计的豁免机制。

**核心原则**：
- 豁免**不是修改规则**，规则永远不变
- 豁免**必须来自用户 Mission**，AI 不能自授权
- 每次豁免**仅消耗一次**，用完即失效
- 所有豁免**必须记录日志**，形成审计链

## 2. 权限等级

| 级别 | 含义 | 示例 | 豁免方式 |
|------|------|------|---------|
| **P0 禁止豁免** | 安全底线，永不允许绕过 | 泄露 secret、删除保护分支、提交 .env | 永不允许 |
| **P1 用户确认豁免** | 工程操作，需用户明确授权 | 修改 remote、force push、重写历史 | 用户 Mission 确认后一次性豁免 |
| **P2 自动允许** | 普通操作，无限制 | 更新 docs、调整代码、修改配置 | 无限制 |

### P0 规则清单（永不可豁免）

| 规则 | 内容 |
|------|------|
| C1 | 不得提交 .env、secrets/、credentials/、密钥文件 |
| C5 | 不得删除 main/master 分支 |
| C6 | 不得修改已配置远程（需 P1 豁免） |
| 安全 | 不得泄露密钥、token、密码 |

## 3. 豁免流程

```
Agent 执行任务
    ↓
Policy Conflict Detection
    │  检测到 Mission 与 Policy 冲突
    ↓
生成 Exemption Request
    │  包含：冲突规则、请求操作、理由
    ↓
呈现给用户确认
    │  "检测到任务与 C6 冲突。是否授权临时豁免？"
    ↓
用户确认
    ├── 是 → 记录 Decision Log → 临时豁免执行 → 标记 consumed
    └── 否 → 阻止执行，返回冲突信息
    ↓
执行操作
    ↓
记录 Authorization Event
    │  .project/audit/authorization-events.jsonl
    ↓
豁免 consumed
```

## 4. 豁免记录格式

位置：`.project/decisions/exemptions.md`

```markdown
## EX-2026-001

- **rule**: C6
- **conflict**: 禁止修改 remote
- **requested_action**: 迁移 origin 到 GitHub
- **authorized_by**: user
- **timestamp**: 2026-07-14T08:40:00
- **scope**: current-project, remote migration
- **expiration**: once
- **status**: consumed
- **reason**: 用户明确要求迁移仓库到新 GitHub 组织
```

## 5. Git Hook 集成

Hook 检测到违规时：

```
正常流程：
  检测违规 → 阻止

豁免流程：
  检测违规
    ↓
  检查 .project/decisions/exemptions.md
    ↓
  ├── 存在 active EX-xxx → 输出 Warning + 豁免信息 → 允许通过 → 标记 consumed
  └── 不存在 → 阻止
```

## 6. 审计日志

所有豁免操作写入：

```
.project/audit/authorization-events.jsonl
```

格式：

```json
{"time":"2026-07-14T09:00:00","exemption_id":"EX-2026-001","rule":"C6","action":"remote_migration","agent":"codex","result":"granted","consumed":true}
```

## 7. 变更控制

- 豁免规则本身不可豁免
- 权限等级调整需版本升级
- 审计日志不可删除