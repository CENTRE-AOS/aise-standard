# AISE Audit Trail

> 版本：v1.1.0-beta.5
> 状态：Beta
> 适用范围：所有项目

## 1. 总则

治理行为应该成为项目资产。Git history 记录了代码变更，但不记录治理决策。Audit Trail 确保所有 AISE 治理事件可追溯。

## 2. 日志结构

```
.project/audit/
├── gate-events.jsonl          # Hook 执行记录
├── agent-events.jsonl         # Agent 行为记录
└── authorization-events.jsonl # 豁免授权记录
```

## 3. gate-events.jsonl — Hook 执行记录

每条记录对应一次 Hook 执行：

```json
{"time":"2026-07-14T09:00:00","gate":"pre-commit","agent":"deepseek-v4","branch":"main","files":3,"result":"pass","duration_ms":120}
{"time":"2026-07-14T09:05:00","gate":"pre-push","agent":"deepseek-v4","branch":"feature/gate","result":"blocked","reason":"force_push","policy":"C6","duration_ms":85}
{"time":"2026-07-14T09:06:00","gate":"pre-push","agent":"deepseek-v4","branch":"feature/gate","result":"pass","exemption":"EX-2026-001","duration_ms":90}
```

字段说明：

| 字段 | 说明 |
|------|------|
| `time` | ISO 8601 时间戳 |
| `gate` | Hook 名称（pre-commit / commit-msg / pre-push / post-merge） |
| `agent` | 执行 Agent 模型 |
| `branch` | 当前分支 |
| `files` | 涉及文件数（仅 pre-commit） |
| `result` | pass / blocked / warning |
| `reason` | 阻止原因（仅 blocked） |
| `policy` | 触发规则（仅 blocked） |
| `exemption` | 豁免 ID（仅豁免通过） |
| `duration_ms` | 执行耗时（毫秒） |

## 4. agent-events.jsonl — Agent 行为记录

记录 Agent 的治理相关操作：

```json
{"time":"2026-07-14T09:00:00","agent":"deepseek-v4","action":"bootstrap","aise_version":"1.1.0-beta.2","result":"pass"}
{"time":"2026-07-14T09:15:00","agent":"deepseek-v4","action":"commit","type":"feat","scope":"git-governance","files":5,"result":"pass"}
{"time":"2026-07-14T09:30:00","agent":"deepseek-v4","action":"archive","version":"1.1.0-beta.2","result":"pass"}
```

## 5. authorization-events.jsonl — 豁免授权记录

```json
{"time":"2026-07-14T09:05:00","exemption_id":"EX-2026-001","rule":"C6","action":"remote_migration","agent":"deepseek-v4","authorized_by":"user","result":"granted","consumed":true}
```

## 6. 日志生命周期

- 日志文件追加写入，永不删除
- 文件达到 10MB 时自动轮转：`gate-events-2026-07.jsonl`
- 日志属于项目资产，随 Git 提交
- 日志文件排除在 `.gitignore` 之外

## 7. 查询示例

```bash
# 查看所有阻止事件
grep '"result":"blocked"' .project/audit/gate-events.jsonl

# 查看所有豁免使用
grep '"exemption"' .project/audit/gate-events.jsonl

# 统计各 Agent 操作次数
grep -o '"agent":"[^"]*"' .project/audit/agent-events.jsonl | sort | uniq -c
```