# Redundancy and Overlap Register

Status: draft v0

## Candidate overlaps

| ID | Concepts | Status | Question |
|---|---|---|---|
| OVR-001 | omega_r / optionality / recoverability / field integrity | REQUIRES_VERIFICATION | Are lab FI metrics measuring recoverability margin? |
| OVR-002 | TPI / semantic elasticity / curvature continuity | REQUIRES_VERIFICATION | Are lab elasticity controls enforcing transformability or only visualising it? |
| OVR-003 | residue / manifold pressure / curvature stress | REQUIRES_VERIFICATION | Are these one pressure mechanism or distinct signals? |
| OVR-004 | DecisionRecord / audit record / receipt | PARTIALLY_RESOLVED | admissibility-kernel has decision records, not certified receipts |
| OVR-005 | RVK branch contamination / field integrity | REQUIRES_VERIFICATION | Are branch integrity and field integrity separable? |
| OVR-006 | SAFE_HOLD / DEFER | ACTIVE_GUARD | SAFE_HOLD must map to DEFER in public API unless API spec changes |

## Current ruling

admissibility-kernel owns canonical public-layer invariant enforcement.

Lab repos may own signal generation, telemetry, and projection loops.

Runtime/proof repos may own receipts, replay, provenance, lineage, and hash-chain proof surfaces.

Do not collapse these layers.

## OVR-007 Gate implementation overlap

| Field | Finding |
|---|---|
| Concepts | admissibility-kernel AdmissibilityGate / runtime components gate |
| Status | ACTIVE_AUDIT |
| Current ruling | Not duplicate; likely formal kernel vs concrete runtime specialization |
| Shared semantics | projected/predicted state, EXECUTE, TRANSFORM, DEFER, REJECT, reasoned routing |
| Runtime-specific semantics | surface class, coherence relations, horizon budget, HBTL trigger, reshape required |
| Kernel-specific semantics | C1-C7, TPI, RVK, DeferRecord, TransformRecord |
| Promotion trap | Do not claim one supersedes the other until full comparison is complete |
