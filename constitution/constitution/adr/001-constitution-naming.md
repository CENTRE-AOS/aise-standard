# Architecture Decision Record: Constitution Naming

> ADR-001
> 日期: 2026-07-17
> 状态: Recorded（待 Phase 3 解决）
> 影响: Protocol Layer, Runtime Layer

---

## 语境

当前 CENTRE 生态中存在两个"Constitution"：

| 位置 | 文件 | 角色 |
|------|------|------|
| aise-standard (Protocol Factory) | Protocols/CONSTITUTION.md | Protocol Constitution — 定义世界规则 |
| agent-governance (Runtime) | constitution/CONSTITUTION.md | Runtime Constitution — 定义执行规则 |

这两个文件内容不同，但命名相同。Agent 或开发者看到 `constitution/` 时无法立即区分"这是协议层的宪法还是运行时的宪法"。

## 决策

暂不修改。在 Phase 3 前需解决。

## 建议方案

```
Protocol Layer:
  AISE Constitution          ← 最高规则（来自 Protocol Factory）
  
Runtime Layer:
  CENTRE Runtime Policy      ← 执行规则（来自 Runtime）
```

或者：

```
aise-standard:
  Protocols/CONSTITUTION.md  →  保持不变（这是协议宪法）

agent-governance:
  constitution/CONSTITUTION.md → 重命名为 RUNTIME_POLICY.md
  或 constitution/ → runtime-policy/
```

## 后果

- 当前不阻塞 Phase 2 冻结
- 不阻塞 Phase 3 开发
- 但 Phase 3 前必须解决，否则 Agent 语义歧义会扩散
- 建议在 Phase 3 Tenant Identity 设计时同步解决（因为 Identity 层需要明确"谁的最高规则"）

## 相关

- `aise-standard/Protocols/CONSTITUTION.md`
- `agent-governance/constitution/CONSTITUTION.md`
- `agent-governance/constitution/contracts/runtime-contract.md`