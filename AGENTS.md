# AGENTS.md — Protocol Authority

> Repository: aise-standard
> Role: CENTRE Protocol Specification
> Protocol: 2.0.0-frozen

---

## Before Any Action

**You MUST read these files in order before making any change:**

1. `AGENT_CONTEXT.md` — Repository identity and authority ranking
2. `VERSION` — Current protocol version
3. `.agent/bootstrap.md` — Bootstrap Descriptor

---

## What You Can Modify

- `protocol/` — Protocol definitions (Contracts, Schemas, Governance Rules)
- `registry/` — Registry entries
- `templates/` — Project templates
- `docs/` — Documentation (non-frozen)

---

## What You CANNOT Modify

- `constitution/CONSTITUTION.md` — Constitution v1.0.0frozen (historical artifact, immutable)
- `constitution/FOUNDATION_FREEZE.md` — Foundation Freeze v3.0.0 (immutable)
- Any file tagged with `frozen` in its version header
- Protocol modifications require RFC Process, not direct Agent edits

---

## Authority Boundaries

| You Own | You Do NOT Own |
|---------|---------------|
| Protocol definitions | Runtime execution |
| Constitution | Agent state |
| Contracts | Build logic |
| Schemas | Installation logic |
| Governance rules | Project context |

---

## Forbidden Actions

- Do NOT modify frozen constitution documents
- Do NOT modify historical AISE 1.x artifacts
- Do NOT add runtime implementation code
- Do NOT add build scripts or CI/CD configs

---

## Context Rules

- `docs/AISE-v1.2-*.md` files are historical proposals, not current truth
- `constitution/` version 1.0.0frozen is the Constitution version, independent of Protocol version 2.0.0-frozen
- This repository defines rules; it does not execute them