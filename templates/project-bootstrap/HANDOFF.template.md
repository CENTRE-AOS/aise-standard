---
generated: {iso_8601_timestamp}
agent: {ai_model_name}
schema_version: 3.1

## Mission
{任务目标，一句话}

## Progress
- {已完成步骤1}
- {已完成步骤2}

## Blocker
symptom: {具体现象}
failed_attempts:
  1. {方案A — 失败原因}
  2. {方案B — 失败原因}

## Decision Log
1. 决策：{做了什么选择}
   排除：{考虑过但放弃的方案 + 放弃原因}
   状态：{已执行 / 搁置 / 待验证}

2. 决策：{另一个选择}
   排除：{放弃了什么}
   状态：{已执行 / 搁置 / 待验证}

## Key Files
- `{path/to/file1}` — {修改摘要}
- `{path/to/file2}` — {修改摘要}

## Error Log
{粘贴最后错误，或 "No error. Logic/Architecture stuck."}

## Environment Snapshot
branch: {git branch --show-current}
python: {python --version}
venv: {VIRTUAL_ENV 路径或 "none"}
last_commit: {git log -1 --oneline}

## Working State
### Dirty Files
{git status -s 输出，如为空则写 "working tree clean"}

### Uncommitted Changes Summary
{git diff --stat 输出，如为空则写 "no uncommitted changes"}

### Recent Conversation
{基于对话历史，3-5行摘要}

## Next Steps (AI-Inferred)
1. {优先级最高：直接解决 Blocker 的第一步}
2. {次优先：Blocker 解决后的下一步}
3. {后续：相关的优化/扩展}
4. {可选：值得探索的替代方向}

## Test Status
latest: {从 commit 提取的 [test:X/Y] 或 "无测试记录"}
command: {推断的测试命令，如 "pytest tests/ -x"}

## Notes
{可选：值得尝试的新方向、直觉判断、未验证的假设}