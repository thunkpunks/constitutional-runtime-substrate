# Gate Comparison Matrix

Status: draft v0

| Concept | admissibility-kernel | runtime gate | Relation |
|---|---|---|---|
| State input | CurrentState | FieldState | analogous |
| Future-state model | ProjectedState | predicted FieldState | analogous |
| Outcome enum | EXECUTE / TRANSFORM / DEFER / REJECT | EXECUTE / TRANSFORM / DEFER / REJECT | aligned |
| Optionality floor | omega_r / C4 | Omega floor | aligned |
| Renegotiability | theta / C2 | Theta floor + tau cooling | aligned |
| Commitment accumulation | residue | tau / horizon budget | partial overlap |
| Transformability | TPI | coherence + reshape + horizon/surface checks | partial overlap |
| Branch isolation | B_iso / C6 | surface/coherence predicates | partial overlap |
| Defer | DeferRecord / residue | HBTL, horizon exhausted, theta floor | aligned role, different mechanisms |
| Transform | TransformRecord / reframing | RESHAPE_REQUIRED | aligned role |
| Reject | invariant violation | hard NSV/coherence/surface violation | aligned role |
| Replay/receipt | not built in kernel | substrate layers | separate owner |

## Current ruling

admissibility-kernel is the public/formal invariant kernel.

runtime/components/gate.py is the concrete runtime gate specialization over FieldState.

They are not currently redundant, but they overlap semantically.

## Active unresolved question

Is admissibility-kernel intended to be:
1. public abstraction of runtime gate,
2. extracted reusable kernel,
3. replacement for runtime gate,
4. parallel formal testbed?

Until answered, neither gate should be retired.
