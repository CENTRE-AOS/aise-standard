# Archive Manifest

> Date: 2026-07-21
> Phase: S2.11 Root Agent Retirement Cleanup
> Purpose: Mark archived documents as read-only historical record — NOT active agent context.

---

## Archive Policy

1. Documents in `archive/` are **historical records only**.
2. They SHALL NOT be loaded as Agent Context during Bootstrap.
3. They SHALL NOT influence runtime decisions.
4. They MAY be referenced for historical understanding only.

## Archived Documents

| Document | Reason Archived |
|----------|----------------|
| `AISE-v1.2-Architecture-Proposal.md` | Pre-separation architecture proposal (monorepo era) |
| `AISE-Governance-Refactoring-Plan.md` | Pre-separation governance refactoring (monorepo era) |
| `CAPABILITY-IMPACT-AUDIT-REPORT.md` | Phase 12 audit (aos-runtime, imported copy) |
| `CAPABILITY-IMPACT-AUDIT-FINAL.md` | Phase 12 audit final (aos-runtime, imported copy) |
| `P12-B5-RUNTIME-BOUNDARY-VERIFICATION.md` | Phase 12 boundary verification (aos-runtime only) |

## Active Documents (NOT archived)

All documents in `protocol/` and root-level governance documents remain active and authoritative.
