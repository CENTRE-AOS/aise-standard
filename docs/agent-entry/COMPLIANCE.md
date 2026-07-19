# AISE Compliance Check

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

Agent 加载完成 AISE 后，必须输出合规检查报告，完成握手协议。Compliance Check 是 Bootstrap Protocol 的最后一步，确保所有治理组件正确加载。

## 2. 检查流程

```
Agent 加载 AISE
    ↓
读取 Registry/version.json
    ↓
读取 Registry/compliance.json
    ↓
更新 compliance.json 为当前加载状态
    ↓
输出合规检查报告
    ↓
等待用户确认开始工作
```

## 3. compliance.json 格式

```json
{
    "standard": "AISE",
    "version": "2.5.0frozen",
    "schema_version": "1",
    "status": "bootstrapping",
    "bootstrap_completed": false,
    "contracts_loaded": {
        "engineering-contract": false,
        "metadata-contract": false,
        "repository-contract": false,
        "handoff-protocol": false
    },
    "skills_loaded": {
        "aise-bootstrap": false, "aise-inject": false,
        "archive": false, "fetch": false, "gitops": false,
        "handoff": false, "publish": false, "review": false,
        "sync": false, "verify": false
    },
    "policies_loaded": {
        "security-policy": false,
        "git-policy": false,
        "rollback-policy": false,
        "exemption-policy": false,
        "audit-policy": false,
        "upgrade-policy": false
    },
    "git_governance_loaded": false,
    "mission_boundary_loaded": false,
    "agent_identity_loaded": false,
    "adr_loaded": false,
    "upgrade_policy_loaded": false,
    "mission_boundary_enabled": false,
    "exemption_enabled": false,
    "audit_enabled": false,
    "memory_ownership_loaded": false,
    "memory_structure_valid": false,
    "started_at": "<ISO 8601 timestamp>",
    "completed_at": "<ISO 8601 timestamp>",
    "agent_model": "<ai-model-name>",
    "project_path": "<project-abs-path>"
}
```

## 4. 合规检查报告模板

```
=== AISE Compliance Check ===

Standard: AISE
Version: v2.5.0frozen (Frozen)
Schema: v1

Loaded:
  Contracts:        [✓/✗] (4/4)
  Skills:           [✓/✗] (10/10)
  Policies:         [✓/✗] (10/10)
  Registry:         [✓/✗]
  Git Gov:          [✓/✗] (7 gates)
  Mission Boundary: [✓/✗]
  Agent Identity:   [✓/✗]
  ADR:              [✓/✗]
  Upgrade Policy:   [✓/✗]
  Memory Ownership: [✓/✗]
  Memory Structure: [✓/✗]

Mode: Autonomous Agent Engineering
Agent Model: <model-name>

Governance Status:
  Exemption: [Enabled/Disabled]
  Audit:     [Enabled/Disabled]

Compliance: [PASS / FAIL]

If FAIL:
  - Missing: <list missing>
  - Action: Fix manually or re-bootstrap

=== End ===
```

## 5. 状态定义

| 状态 | 说明 |
|------|------|
| frozen | 稳定版本，可生产使用 |
| beta | 测试版本，功能完整待验证 |
| proposal | 提案版本，用于技术讨论 |
| deprecated | 已废弃，不推荐使用 |

## 6. 变更控制

进入 Frozen 后：
- 允许新增检查项
- 禁止修改状态流转顺序