# Handoff Protocol

> 版本：v1.0
> 状态：Frozen
> 适用范围：所有项目（Repository Generic）

## 1. 总则

Handoff Protocol 定义 AI Agent 之间的交接标准。任何符合本协议的 Agent（Codex、Claude Code、Trae、Workbench Agent 等）均可无缝接续。

## 2. 触发条件

以下场景必须生成 HANDOFF.md：

- Cloud Workspace 销毁前
- 当前任务需要暂停并由另一个 AI 接续
- 用户明确要求移交

## 3. HANDOFF.md 位置

```text
<project>/.handoff/HANDOFF.md
```

## 4. 必填字段

| 字段 | 类型 | 说明 |
|------|------|------|
| generated | ISO 8601 | 生成时间戳 |
| agent | string | 当前 AI 模型名 |
| schema_version | string | 协议版本号 |
| Mission | string | 任务目标 |
| Progress | array | 已完成步骤 |
| Blocker | object | 当前阻塞点 |
| Blocker.failed_attempts | array | 已尝试的失败方案 |
| Decision Log | array | 关键决策记录 |
| Key Files | array | 关键文件路径 |
| Error Log | string | 最近错误 |
| Environment Snapshot | object | 环境信息 |
| Working State | object | 未保存修改、运行进程 |
| Recent Conversation | string | 对话摘要 |
| Next Steps (AI-Inferred) | array | AI 推断的下一步 |
| Test Status | object | 测试状态 |
| Notes | string | 补充说明 |

## 5. Decision Log 格式

每条决策记录必须包含：

```markdown
1. 决策：[做了什么选择]
   原因：[为什么选这个]
   排除：[考虑过但放弃的方案 + 放弃原因]
   状态：[已执行 / 搁置 / 待验证]
```

## 6. 接替流程

接替 AI 按以下顺序读取：

1. `.handoff/HANDOFF.md` — 交接文档（核心）
2. `PROJECT_BLUEPRINT.md` — 项目蓝图
3. `git log --oneline -15` — 最近变更历史
4. `AISE/Registry/skills.json` — 已注册技能

## 7. Decision Log 约束

- 接替 AI 严格不重复 failed_attempts 中的方案
- 优先查看 Key Files，理解当前改动
- 如有未提交改动，先确认是否 git stash
- 如有测试命令，先跑测试确认基线

## 8. 增量采集

初次移交后，后续移交必须增量采集：

- 从上一次 HANDOFF 的 generated 时间戳开始
- 只采集新产生的 Decision Log
- 保留旧 HANDOFF 的完整历史归档

## 9. 变更控制

进入 Frozen 后：

- 允许新增可选字段。
- 禁止删除必填字段或改变其语义。