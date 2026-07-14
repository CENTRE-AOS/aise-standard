# Rollback Policy (C3 Recovery Protocol)

> 版本：v1.0
> 状态：Frozen
> 适用范围：所有项目（Repository Generic）

## 1. 总则

任何操作失败必须根据操作阶段选择正确回滚策略，禁止继续错误操作。本协议定义分阶段回滚规则。

## 2. Recovery Protocol

### Before Commit（尚未提交）

操作还没提交到本地仓库。

```bash
# 回滚暂存区
git reset --hard HEAD

# 恢复备份文件
if [ -f "FILE.md.bak" ]; then
    mv FILE.md.bak FILE.md
fi
```

### After Commit（已提交本地，但尚未推送）

提交已进入本地 Git，但未推送至远程。

```bash
# 回滚最后一个提交
git revert HEAD
# 或 git reset --hard HEAD~1（如果还没推送）
```

### After Tag（已打标签，但尚未推送）

标签已创建本地，但未推送至远程。

```bash
# 删除本地标签
git tag -d "v<version>"

# 回滚提交（如果还没推送）
git reset --hard HEAD~1

# 恢复备份文件
# ...
```

### After Push（已推送至远程）

代码已推送，不能重写历史。

```bash
# 创建修复 commit 回滚变更
git revert <commit>
```

**禁止**：

- 禁止历史重写
- 禁止强制推送
- 禁止删除远程标签

只能通过新增 revert commit 回滚。

## 3. 回滚流程

```
1. 确定失败阶段
2. 按阶段规则执行回滚
3. 确认回滚完成（git status → clean）
4. 报告失败原因和回滚结果
5. 等待用户确认下一步
```

## 4. 错误处理

| 异常场景 | 处理 |
|---------|------|
| 回滚失败 | 终止，报告回滚失败 |
| `.bak` 不存在 | 无法恢复原始文件，报告缺失 |
| 回滚后工作区不干净 | 清理 untracked 文件（用户确认后） |

## 5. 变更控制

进入 Frozen 后：

- 允许新增阶段（如 After Merge）。
- 禁止放宽回滚约束。