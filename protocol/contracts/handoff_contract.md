# Handoff Contract

> 版本：v1.0
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

本合约定义 Handoff 的标准格式与内容，确保 AI 之间的上下文交接完整、可理解、可接续。

## 2. Handoff 触发条件

以下场景必须生成 `HANDOFF.md`：

- Cloud Workspace 销毁前
- 当前任务需要暂停并由另一个 AI 接续
- 用户明确要求移交

## 3. HANDOFF.md 位置

默认位于项目根目录：

```text
<project>/HANDOFF.md
```

## 4. 必填字段

| 字段 | 说明 |
|------|------|
| Mission | 当前任务目标 |
| Progress | 已完成的工作 |
| Blocker | 当前阻塞点 |
| Decision Log | 已做出的关键决策 |
| Pending Questions | 等待用户确认的问题 |
| Key Files | 关键文件路径 |
| Error Log | 最近的错误与处理状态 |
| Environment Snapshot | 环境信息 |
| Working State | 未保存修改、运行中的进程 |
| Recent Conversation | 最近对话摘要 |
| Next Steps | 明确的下一步行动 |
| Test Status | 测试状态 |
| Notes | 其他补充说明 |

## 5. 提交要求

- Handoff 生成的 `HANDOFF.md` 推荐提交到 Git。
- Cloud Workspace 销毁前必须提交并 Push origin。

## 6. 接替流程

接替 AI 应按以下顺序读取：

1. `HANDOFF.md`
2. `docs/engineering-workflow.md`
3. `.project/`
4. `PROJECT_BLUEPRINT.md`
5. `CHANGELOG.md`

## 7. 变更控制

本合约进入 Frozen 状态后：

- 允许新增可选字段。
- 禁止删除必填字段或改变其语义。
