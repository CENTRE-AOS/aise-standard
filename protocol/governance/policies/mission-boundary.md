# Mission Boundary Contract

> 版本：v1.1.0-beta.5
> 状态：Beta
> 适用范围：所有项目

## 1. 总则

Autonomous Agent 在收到用户 Mission 后，必须确认任务边界，防止过度执行。Mission Boundary Contract 定义 Agent 的执行范围约束。

## 2. 核心原则

- Agent 只能修改 Mission 范围内允许的文件
- Agent 不能扩大修改范围
- Agent 不能自动触发额外任务（如重构、依赖更新、部署变更）

## 3. 边界文件

### scope.json

```json
{
    "task": "fix-login-bug",
    "description": "修复登录模块的认证超时问题",
    "allowed_paths": [
        "src/auth/",
        "tests/auth/"
    ],
    "forbidden_paths": [
        "database/",
        "deployment/",
        "src/payment/",
        "config/production/"
    ],
    "max_files_changed": 20,
    "max_lines_changed": 500,
    "can_modify_dependencies": false,
    "can_modify_config": false,
    "can_modify_tests": true,
    "requires_review": true
}
```

### constraints.json

```json
{
    "cannot": [
        "修改数据库 schema",
        "修改部署配置",
        "添加新依赖",
        "修改 .env 文件",
        "修改 Git remote 配置",
        "删除任何文件（除非确认）"
    ],
    "must": [
        "每次修改后运行相关测试",
        "更新 CHANGELOG.md",
        "commit message 格式符合 AISE 标准",
        "修改前 git diff 确认范围"
    ]
}
```

## 4. 边界检查流程

```
Agent 收到 Mission
    ↓
读取 .project/mission/scope.json
    ↓
    ├── 存在 → 按边界约束执行
    └── 不存在 → 提示用户定义边界
    ↓
每次修改前：
    ├── 检查文件是否在 allowed_paths
    ├── 检查文件是否在 forbidden_paths
    ├── 检查变更量是否超出限制
    └── 超出 → 阻止并提示用户
    ↓
Mission 完成 → 更新 scope.json 状态
```

## 5. 违规处理

Agent 检测到超出边界时：

```
⚠  Mission Boundary Violation

File: src/database/migration.py
Reason: 不在 allowed_paths 中
Mission scope: src/auth/, tests/auth/

This file is outside the current mission boundary.
如果确实需要修改此文件，请更新 Mission Scope。
```

## 6. 审计

所有边界违规记录写入 `.project/audit/agent-events.jsonl`。

## 7. 变更控制

- 边界由用户 Mission 定义，AI 不可自定
- 边界可随时由用户调整