# AISE Upgrade Protocol

> 版本：v2.0.0
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

AISE 版本升级必须遵循安全协议，确保不破坏项目、不覆盖用户修改、不污染版本历史。

## 2. 升级流程

```
检测新版本（AISE/Registry/version.json vs 远程）
    ↓
差异分析
    ├── 新增文件
    ├── 修改文件
    └── 删除文件
    ↓
生成升级计划
    ↓
备份（创建 backup tag）
    ↓
执行迁移（AISE/Migrations/<from>_to_<to>.md）
    ↓
验证（aise verify）
    ↓
更新 VERSION
    ↓
提交
```

## 3. 迁移脚本格式

位置：`AISE/Migrations/1.0.0_to_1.1.0.md`

```markdown
# AISE Migration: 1.0.0 → 1.1.0

## 升级前检查
- [ ] 工作区干净（git status）
- [ ] 已创建备份 tag
- [ ] 所有测试通过

## 变更摘要
- 新增 Git-Governance/ 目录
- 新增 Policies/exemption-policy.md
- 新增 Policies/audit-policy.md
- 修改 SYSTEM.md（五层架构）
- 修改 Bootstrap.md（新增 Step 5）

## 文件变更
### 新增
- AISE/Git-Governance/
- AISE/Policies/exemption-policy.md
- AISE/Policies/audit-policy.md

### 修改
- AISE/SYSTEM.md
- AISE/Agent-Entry/Bootstrap.md
- AISE/Registry/version.json

### 删除
- 无

## 回滚方案
```bash
git checkout <backup-tag> -- AISE/
git commit -m "revert: rollback AISE to v1.0.0"
```

## 升级后验证
- [ ] aise verify 通过
- [ ] aise install-hooks 执行
- [ ] git commit 测试
- [ ] git push 测试
```

## 4. 兼容性检查

升级前必须检查：

```json
{
    "compatibility": {
        "contracts": {
            "engineering-contract": "compatible",
            "metadata-contract": "compatible",
            "repository-contract": "compatible",
            "handoff-protocol": "compatible"
        },
        "policies": {
            "security-policy": "compatible",
            "git-policy": "backward_compatible",
            "rollback-policy": "compatible",
            "exemption-policy": "new",
            "audit-policy": "new"
        },
        "skills": {
            "all": "compatible"
        }
    }
}
```

## 5. 安全约束

- 禁止自动升级（需用户确认）
- 升级前必须创建备份 tag
- 升级失败自动回滚
- 禁止跨大版本跳级（如 1.0 → 2.0 需先 1.0 → 1.1 → 1.2 → 2.0）

## 6. 变更控制

- 迁移脚本不可回退执行
- 已执行的迁移脚本不可删除