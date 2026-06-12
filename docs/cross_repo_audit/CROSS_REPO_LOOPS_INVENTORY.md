# Cross-Repo Loops Inventory

Status: draft v0

## Built loops in admissibility-kernel

| ID | Loop | Owner repo | Status |
|---|---|---|---|
| LOOP-001 | Proposal -> ProjectedState | admissibility-kernel | BUILT |
| LOOP-002 | ProjectedState -> C1-C7 pass/fail | admissibility-kernel | BUILT, with C1/C3 light enforcement caveat |
| LOOP-003 | ProjectedState -> TPIResult | admissibility-kernel | BUILT |
| LOOP-004 | Projection + TPI -> EXECUTE/TRANSFORM/DEFER/REJECT | admissibility-kernel | BUILT |
| LOOP-005 | DEFER -> DeferRecord -> residue pressure | admissibility-kernel | BUILT |
| LOOP-006 | TRANSFORM -> TransformRecord -> omega_r preservation | admissibility-kernel | BUILT |
| LOOP-007 | SessionTrace -> RVK failure modes -> ViabilityReport | admissibility-kernel | BUILT |
| LOOP-008 | EXECUTE -> ObservedState validation | caller / not internal | RECORDED_NOT_BUILT |

## Lab loops to inspect

| Candidate loop | Status |
|---|---|
| Ableton macro -> OSC -> Gate receiver | REQUIRES_VERIFICATION |
| Ableton -> CSV logger | PARTIALLY_BUILT / runtime unverified |
| Logger CSV -> Blender projection | RECORDED_NOT_BUILT transform gap |
| CSV -> Blender visual graph | PARTIALLY_BUILT / runtime unverified |
| FI metric loop | REQUIRES_VERIFICATION |

## Runtime/proof loops to inspect

| Candidate loop | Status |
|---|---|
| candidate -> receipt | REQUIRES_VERIFICATION in this audit |
| receipt -> replay | REQUIRES_VERIFICATION in this audit |
| event -> hash chain | REQUIRES_VERIFICATION in this audit |
| transformation -> lineage | REQUIRES_VERIFICATION in this audit |
