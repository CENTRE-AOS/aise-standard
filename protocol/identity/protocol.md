# Identity Protocol

> **版本**: 1.0  
> **状态**: Frozen  
> **所属协议族**: AISE Protocol 1.0

## 1. 目的

回答 Agent 进入项目时的第一个问题：

> **Who am I?**

Identity Protocol 定义 Agent 如何识别一个项目、定位项目、确认项目的治理关系。

## 2. 核心原则

- 每个项目必须有唯一身份。
- 项目身份不依赖 Agent、IDE 或平台。
- 身份由项目声明，Agent 只读取。

## 3. 协议声明

项目通过 `.agent-entry.json` 声明自身身份：

```json
{
  "protocol": "AISE",
  "version": "1.0",
  "project_id": "uuid-or-project-identity",
  "project": {
    "version": "1.0.0"
  },
  "governance": {
    "provider": "agent-governance",
    "runtime": "2.5.0frozen"
  }
}
```

### 三层版本模型

| 层级 | 字段 | 说明 | 示例 |
|------|------|------|------|
| Project Version | `project.version` | 项目自身版本，SemVer 或项目定义格式 | `1.0.0`, `2.5.0frozen` |
| AISE Protocol | `version` | AISE Protocol 版本，当前为 `1.0` | `1.0` |
| Governance Runtime | `governance.runtime` | Runtime 实现版本，用于兼容性检查 | `2.5.0frozen` |

三者彻底解耦：
- 项目升级到 v5.4.1 无需修改 Runtime。
- Runtime 升级到 v2.6.0 无需修改项目版本。
- 协议版本仅在进行协议级变更时修改。

## 4. 身份资产

### 4.1 Protocol Manifest

- 文件: `.agent-entry.json`
- 所有者: 项目
- 写权限: Protocol Runtime (`aise init` / `aise migrate`)
- 读权限: 所有 Agent

### 4.2 Repository Identity

- 维护者: Governance Runtime
- 内容: 项目 ID、GitHub 仓库、本地路径、默认分支、保护分支
- 存储位置: Runtime Registry 内部（如 `agent-governance/registry/installed_projects.json`），非 AISE 协议标准

### 4.3 Workspace Binding

- 维护者: Governance Runtime
- 内容: 机器绑定、本地路径、Agent 工作区映射
- 存储位置: Runtime Registry 内部，非 AISE 协议标准

## 5. Agent 读取顺序

Agent 进入项目时：

```text
1. 发现 .agent-entry.json
2. 读取 project_id
3. 查询 repository-identities.json 确认项目身份
4. 查询 workspace-bindings.json 确认本地绑定
5. 验证 Git origin / 分支 / 远程
6. 进入 Asset Protocol
```

## 6. Schema

参见: `schema.json`
