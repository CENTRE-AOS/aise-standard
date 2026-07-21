# Authority Boundary Audit

> Version: 1.0.0-frozen
> Date: 2026-07-21
> Phase: S2.8 — Cross Authority Boundary Audit
> Status: FROZEN
> Purpose: Prevent any Project Agent from re-invading another Authority after Root Agent exit.

---

## 1. Five-Authority Boundary

```
A0  Protocol Authority     → aise-standard
        │
        │ defines governance
        ▼
A1  Runtime Source Authority → aos-runtime
        │
        │ provides source (read-only to Factory)
        ▼
A2  Build Authority         → aos-factory
        │
        │ produces signed artifact
        ▼
A3  Artifact Authority      → RESERVED (Phase 3)
        │
        │ stores and distributes
        ▼
A4  Install Authority       → aos-installer
        │
        │ deploys from artifact
        ▼
Runtime Instance
```

---

## 2. Per-Agent Boundary Verification

### A0 Protocol Agent (aise-standard)

```
CAN modify:
    ✅ protocol/rfc/           — RFC documents
    ✅ protocol/schema/        — schema definitions
    ✅ protocol/governance/    — governance rules
    ✅ constitution/           — constitutional documents
    ✅ CHANGELOG.md            — version history
    ✅ PROJECT_BLUEPRINT.md    — architecture state
    ✅ VERSION                 — protocol version

CANNOT modify:
    ❌ aos-runtime/**           — Runtime implementation
    ❌ aos-factory/**           — Factory implementation
    ❌ aos-installer/**         — Installer implementation
    ❌ Any implementation code  — Protocol is governance, not code
```

### A1 Runtime Agent (aos-runtime)

```
CAN modify:
    ✅ runtime/kernel/              — kernel implementation
    ✅ Skills/                      — skill implementations
    ✅ runtime/kernel/*.json        — authority maps
    ✅ runtime/kernel/governance-loop/ — governance loop
    ✅ VERSION                      — runtime version
    ✅ CHANGELOG.md                 — version history
    ✅ PROJECT_BLUEPRINT.md         — architecture state

CANNOT modify:
    ❌ aise-standard/**              — Protocol contracts
    ❌ aos-factory/**                — Factory implementation
    ❌ aos-installer/**              — Installer implementation
    ❌ Protocol frozen contracts     — once frozen, read-only
```

### A2 Factory Agent (aos-factory)

```
CAN modify:
    ✅ factory.manifest.json         — build manifest
    ✅ Build pipeline scripts        — build automation
    ✅ CI/CD configuration           — GitHub Actions
    ✅ Signing logic                 — artifact signing
    ✅ Release workflow              — release automation
    ✅ VERSION                       — factory version
    ✅ CHANGELOG.md                  — version history

CANNOT modify:
    ❌ aise-standard/**              — Protocol contracts
    ❌ aos-runtime/**                — Runtime source
    ❌ aos-installer/**              — Installer implementation
    ❌ Runtime kernel                — not Factory territory
    ❌ Artifact Registry ownership   — A3 is RESERVED
    ❌ Installer logic               — A4 territory
```

### A4 Installer Agent (aos-installer)

```
CAN modify:
    ✅ CLI implementation            — command-line tools
    ✅ Injection system              — artifact injection
    ✅ Deployment logic              — deployment automation
    ✅ Bootstrap engine              — bootstrap protocols
    ✅ VERSION                       — installer version
    ✅ CHANGELOG.md                  — version history

CANNOT modify:
    ❌ aise-standard/**              — Protocol contracts
    ❌ aos-runtime/**                — Runtime source (FORBIDDEN)
    ❌ aos-factory/**                — Factory implementation (FORBIDDEN)
    ❌ Artifact Registry ownership   — A3 is RESERVED
    ❌ Runtime Contract              — A0 territory
    ❌ Factory Interface             — A2 territory
```

---

## 3. Boundary Enforcement Rules

### Rule 1: Source Plane Separation

```
Runtime Source (A1)  ≠  Build Artifact (A2)  ≠  Install Target (A4)
```

No agent may treat one repository's output as another repository's source code reference.

### Rule 2: Release Plane Separation

```
Source (A0/A1) → Build (A2) → Artifact (A3) → Install (A4)
```

Consumption crosses the Release Plane via signed artifacts ONLY, never via git clone or source reference.

### Rule 3: Authority Immutability

Once an authority level is assigned to a repository, it SHALL NOT be changed without Architecture Governor review.

### Rule 4: No Agent Merging

No single agent SHALL hold multiple authority levels simultaneously. One agent = one repository = one authority.

---

## 4. Violation Detection

If any agent performs an operation outside its CAN scope:

1. Operation SHALL be rejected
2. Violation SHALL be logged to audit trail
3. Root Architecture Governor SHALL be notified
4. Session SHALL be terminated if violation is intentional

### Detection Points

| Check | Mechanism |
|-------|-----------|
| File modification outside repo | Git boundary |
| Cross-repo code reference | dependency-lock.json |
| Authority escalation | remote-identity.json verification |
| Protocol modification by non-A0 | bootstrap-contract.md check |

---

## 5. Audit Status

| Authority | Agent | Boundary | Violations |
|-----------|-------|:--------:|:----------:|
| A0 Protocol | protocol-agent | FROZEN | 0 |
| A1 Runtime | runtime-agent | FROZEN | 0 |
| A2 Build | factory-agent | FROZEN | 0 |
| A4 Install | installer-agent | FROZEN | 0 |

**Root Agent certifies**: All four authority boundaries are correctly defined with zero ambiguity. No Project Agent has any legitimate reason to cross its authority boundary after this audit.
