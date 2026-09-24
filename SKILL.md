---
name: adversarial-qa
description: >-
  Use for structured product-level adversary tests of YOUR governance gates
  (omit optional auth field, second surface, inverse route, body authority,
  rollback counters). Not exploit PoCs or offensive cyber — matrix of attack ×
  surface × expected refuse × actual. Stop when a hole is found; fix before next
  attack.
---
# Adversarial QA

Structured product-level attack session against **your own** governance surfaces. Goal: prove gates refuse what they should. This is QA on Vorxeo/G-MAR controls — **not** writing exploits, malware, or offensive cyber against third parties.

## Scope boundary (hard)

| Allowed | Forbidden |
|---------|-----------|
| Call your product's APIs/MCP/CLI with adversarial **inputs** you control | Exploit PoCs, shell payloads, third-party targeting |
| Omit fields, swap subjects, hit inverse routes, reset counters via product APIs | Bypass instructions for systems you don't own |
| Record refuse vs allow in a matrix | Step-by-step attack recipes for external use |

If a hole appears, **stop**, file a finding (`fail-closed-review`), fix, Verified-deliver (`verified-delivery`), then continue. Do not chain exploits.

## When this skill applies

- Before claiming a gate is "done".
- After authz/policy PRs; before Approve in `handoff-faber-rigor`.
- When expanding surfaces (new MCP tool, CLI, admin route).
- Paired with **reachability-audit** to list surfaces first.

## Attack catalog (product gates)

Run each applicable row against **every** listed surface.

1. **Omit optional auth field** — drop `org_id` / `scope` / `audience` / similar; expect refuse, not skip.
2. **Second surface** — same mutation via MCP/CLI/job/admin twin; expect same refuse.
3. **Inverse route** — read/list/export path that leaks or mutates without the write-gate.
4. **Body authority** — body `user_id`/`role` disagrees with credential; expect credential wins or refuse.
5. **Rollback counters** — after deny/warn, ensure counters/flags cannot be reset via unauthenticated or lesser-auth path to re-open a window.
6. **Warn-mode honesty** — in warn/not_enforced, response must not claim hard block (`contract-and-compat`).
7. **Gate error path** — force policy engine error (invalid config in test env); expect deny/fail, not allow (`except-as-allow`).

Add product-specific rows as needed; keep names stable in the matrix.

## Matrix format (required)

Produce a table. Do not invent Actual — fill from Verified runs.

```text
| Attack              | Surface           | Expected | Actual  | Result   | Evidence file        |
|---------------------|-------------------|----------|---------|----------|----------------------|
| omit-optional-auth  | POST /v1/x        | refuse   | refuse  | PASS     | /tmp/aqa-01.txt      |
| omit-optional-auth  | MCP tool x.create | refuse   | allow   | HOLE     | /tmp/aqa-02.txt      |
| body-authority      | POST /v1/x        | refuse   | ...     | ...      | ...                  |
```

Definitions:

- **Expected refuse** — gate should deny or hard-fail closed.
- **Actual** — from result file only (`verified-delivery`).
- **HOLE** — Actual ≠ Expected → **STOP**.

## Stop-on-hole protocol

```text
1. Mark row HOLE; do not run further attacks on other rows until fixed
   (optional: finish documenting remaining Planned rows as SKIPPED).
2. Open finding with fail-closed-review format (severity, evidence, why fail-open, required fix).
3. Faber implements fix; Verified delivery on the failing row.
4. Re-run the HOLE row; then continue the matrix from the next Planned attack.
```

**Failure mode spray-and-pray**: run all attacks, dump many holes, fix none — forbidden. One hole → fix → resume.

## Session procedure

1. **Enumerate surfaces** — HTTP routes, MCP tools, gRPC methods, CLI commands, workers (reachability-audit).
2. **Pick attack set** — from catalog; prioritize new/changed gates.
3. **For each cell** — craft minimal adversarial request (product test client); redirect output to result file; Read; fill Actual.
4. **On PASS** — next cell.
5. **On HOLE** — stop-on-hole protocol.
6. **End** — matrix complete with no open HOLEs; attach to handoff packet.

## What "refuse" means

Document per product, but typically:

- Authz deny with stable error code (not 500).
- No side effect (no write, no counter reset, no dual-write).
- Mode-honest body if warn (would_deny, not blocked).

Side-effect check is part of Actual — a 403 that still wrote is a HOLE.

## Failure modes (named)

| Name | Meaning |
|------|---------|
| **omit-to-bypass** | Optional field missing → allow |
| **second-door** | Twin surface allows |
| **inverse-leak** | Read/export skips write gate |
| **body-as-principal** | Body subject wins |
| **counter-rollback** | Lesser auth resets enforcement window |
| **enforcement-lie** | Warn reports as blocked |
| **except-as-allow** | Gate error → allow |
| **spray-and-pray** | Many holes, no fix before next attack |
| **PoC-creep** | Session drifts into offensive exploit writing — abort |

## Interaction with other skills

- **fail-closed-review**: findings format; severity.
- **verified-delivery**: every Actual cell cites a result file.
- **contract-and-compat**: front-door parity; not_enforced honesty.
- **migration-and-data-safety**: post-migrate counter/flag attacks.
- **bench-loop**: adversarial QA is not a bench score; do not substitute.
- **handoff-faber-rigor**: matrix (or "NOT tested: adversarial-qa") required for authz PRs.
- **karpathy-method** / **code-that-holds**: fix the motor/gate, not the test harness alone.

## Anti-patterns

- Calling this "penetration testing" and attaching Metasploit-style steps.
- Marking PASS because the happy path works.
- Testing only the HTTP front door.
- Continuing the matrix after a HOLE "to gather more data" without a fix in flight — gather is fine only as SKIPPED documentation, not as an excuse to skip the fix gate.
