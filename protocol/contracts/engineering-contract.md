# Engineering Contract

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有项目（Repository Generic）

## 1. 总则

本合约定义 AI 软件工程的基础流程与约束，确保所有 AI 执行统一的工程标准。

AISE 支持两种协作模式：

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **Autonomous Agent Engineering** | AI 自主完成 Architecture → Implementation → Testing → Review → Release 全流程，用户只需提供 Mission | 目标模式 |
| **AI 辅助开发** | AI 辅助用户完成部分代码修改，用户审核后提交 | 过渡/兼容模式 |

本合约以 Autonomous Mode 为基准定义。

## 2. 工程生命周期

```
GitOps → Architecture → Implementation → Testing → Review → Archive → Publish → Sync → Fetch → Handoff → Run
```

禁止跳步骤。

## 3. 各阶段职责

| 阶段 | 负责 Skill | 关键产出 |
|------|-----------|---------|
| GitOps | gitops | 项目骨架、目录结构、AISE 注入 |
| Architecture | 用户 | 架构设计文档 |
| Implementation | 用户+AI | 代码实现 |
| Testing | 用户+AI | 测试通过 |
| Review | review | Review Report |
| Archive | archive | Commit、Tag、CHANGELOG、BLUEPRINT |
| Publish | publish | Build Artifact、Release Notes |
| Sync | sync | 升级后的 Workspace |
| Fetch | fetch | 最新代码状态 |
| Handoff | handoff | HANDOFF.md |
| Run | 用户 | 运行应用 |

## 4. 质量门禁

开发完成后，Review 阶段必须通过以下检查：

- 代码结构一致
- Contract 符合
- 测试覆盖可接受
- Review Report 无 Critical 问题

## 5. 治理约束

- 遵循 `Policies/security-policy.md`
- 遵循 `Policies/git-policy.md`
- 遵循 `Policies/rollback-policy.md`
- 遵循 `Policies/exemption-policy.md`（P0/P1/P2 豁免）
- 遵循 `Policies/audit-policy.md`（审计追踪）
- 遵循 `Git-Governance/`（Git 出入口控制）
- 遵循 `Policies/mission-boundary.md` — Mission Boundary 合约
- 遵循 `Policies/adr-policy.md` — ADR 架构决策记录
- 遵循 `Policies/agent-identity.md` — Agent 身份与能力声明
- 遵循 `Policies/upgrade-policy.md` — 版本升级协议

## 6. 变更控制

进入 Frozen 后：
- 允许新增阶段
- 禁止调整已有阶段顺序