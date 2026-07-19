# AOS Gate Contract — Protocol Layer

> 定义 Gate 的规范（WHAT），不定义执行（HOW）。执行由 Runtime 层负责。

## 8 道门禁契约

### Gate 0: Identity Check
- **规范**：Agent 进入任何项目前，必须验证 Agent 身份。
- **检查项**：Agent 准入状态、.agent/ 目录、身份声明。
- **失败处理**：终止进入流程。

### Gate 1: Project Admission
- **规范**：Agent 进入 AISE 治理项目时，必须执行 Protocol Manifest 验证。
- **检查项**：.project/centre.protocol.json 存在性、JSON 有效性、字段完整性、protocol_id 匹配、schema 兼容性、协议版本匹配、runtime 版本充足。
- **失败处理**：任一步失败，终止进入流程。

### Gate 2: Protocol Validation
- **规范**：验证协议版本兼容性。
- **检查项**：centre.cert.json、protocol_version、runtime_version 兼容性。
- **失败处理**：终止操作，报告版本不兼容。

### Gate 3: Contract Validation
- **规范**：检查工程合约合规性。
- **检查项**：RFC Contracts、架构边界、API 契约。
- **失败处理**：终止操作，报告合约违规。

### Gate 4: Build Validation
- **规范**：执行构建验证。
- **检查项**：构建成功、测试通过、产物完整性。
- **失败处理**：终止操作，报告构建失败。

### Gate 5: Security Validation
- **规范**：C1-C9+M0 安全基线检查。
- **检查项**：Git 状态、敏感文件保护、历史保护、原子写入。
- **失败处理**：终止操作，触发 C3 Recovery Protocol。

### Gate 6: Release Validation
- **规范**：发布前完整性检查。
- **检查项**：版本号一致性、文档一致性 DG-0、产物 SHA256、Manifest 完整性。
- **失败处理**：终止发布，报告不一致。

### Gate 7: Runtime Publish
- **规范**：Runtime 发布权限验证。
- **检查项**：仅接受 Factory Release Pipeline、签名验证、发布清单校验。
- **失败处理**：拒绝推送，记录审计日志。

## 执行顺序

```
Gate 0 → Gate 1 → Gate 2 → Gate 3 → Gate 4 → Gate 5 → Gate 6 → Gate 7
```

任一步失败，终止后续流程。

## 适用范围

| Gate | Factory | Runtime | AISE Project |
|------|---------|---------|-------------|
| 0-2 | ✓ | ✓ | ✓ |
| 3-5 | ✓ | - | ✓ |
| 6 | ✓ | - | ✓ |
| 7 | ✓ | ✓ | - |