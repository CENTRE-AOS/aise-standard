# Handoff — aise-standard

> Date: 2026-07-21
> Operation: Foundation Integrity Fix — Metadata Consistency Repair
> Branch: fix/foundation-integrity-20260721
> Commit: 300d358

## Modified

| File | Change |
|------|--------|
| registry/version.json | aise_standard: 2.5.0frozen → 2.0.0-frozen; aise_protocol: 1.0 → 2.0.0-frozen; 新增 governance.provider: aos-runtime, compatibility: { runtime_min: 3.2.0, foundation: v3.2.0 } |
| registry/skills.json | aise_version: 2.5.0frozen → 2.0.0-frozen; 14 Skill version: 2.5.0frozen → 3.2.0; 新增 governance.provider: aos-runtime |
| .project/centre.protocol.json | runtime_version: v3.1.0 → v3.2.0; runtime_min_version: v3.1.0 → v3.2.0 |

## Not Modified

- aos-runtime, aos-installer, aos-factory-new — 不属于本 Authority

## Next

- RFC-0003 Production Chain v1.1 — Release Boundary Design（Protocol Layer only）