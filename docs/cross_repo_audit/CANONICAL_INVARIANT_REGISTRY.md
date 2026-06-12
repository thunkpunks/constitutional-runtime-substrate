
## Cross-repo ownership update

| Surface | Canonical owner | Evidence |
|---|---|---|
| Invariant enforcement | admissibility-kernel | Gate, C1-C7, TPI, RVK |
| Decision records | admissibility-kernel | AdmissibilityDecisionRecord, DeferRecord, TransformRecord |
| Receipts | thunkpunks-constitutional_runtime_substrate | receipt tests present |
| Replay | thunkpunks-constitutional_runtime_substrate | replay integration tests present |
| Lineage | thunkpunks-constitutional_runtime_substrate | lineage tests present |
| Evidential loop | thunkpunks-constitutional_runtime_substrate | evidential loop tests present |
| CLI receipt surface | ewt-cli | receipt path references present |

## Revised boundary

admissibility-kernel should not claim canonical proof-surface ownership.

constitutional_runtime_substrate appears to own proof-surface loops:
- receipts
- replay
- lineage
- evidential loop

This requires source inspection before promotion to BUILT.

## Runtime substrate promotion

Promoted after source inspection:

| Surface | Owner repo | Status |
|---|---|---|
| ReceiptLog | thunkpunks-constitutional_runtime_substrate | BUILT |
| Receipt self-hash verification | thunkpunks-constitutional_runtime_substrate | BUILT |
| EvidentialBundle | thunkpunks-constitutional_runtime_substrate | BUILT |
| replay_evidential_chain | thunkpunks-constitutional_runtime_substrate | BUILT |
| assert_replayable | thunkpunks-constitutional_runtime_substrate | BUILT |

Boundary preserved:

- replay validates evidence, not admissibility
- outcome travels as opaque evidence
- replay imports no kernel
- integrator adds no new integrity primitive

## Event log promotion

Promoted after source inspection:

| Surface | Owner repo | Status |
|---|---|---|
| append-only event trace | thunkpunks-constitutional_runtime_substrate | BUILT |
| hash-chain integrity | thunkpunks-constitutional_runtime_substrate | BUILT |
| strict session ordering | thunkpunks-constitutional_runtime_substrate | BUILT |
| value-isolated payload copy | thunkpunks-constitutional_runtime_substrate | BUILT |
| receipt verification against log | thunkpunks-constitutional_runtime_substrate | BUILT |
| tamper detection | thunkpunks-constitutional_runtime_substrate | BUILT |

## Lineage promotion

Promoted after source inspection:

| Surface | Owner repo | Status |
|---|---|---|
| LineageGraph | thunkpunks-constitutional_runtime_substrate | BUILT |
| parent/child ancestry | thunkpunks-constitutional_runtime_substrate | BUILT |
| transitive ancestry reconstruction | thunkpunks-constitutional_runtime_substrate | BUILT |
| event-seq replay order | thunkpunks-constitutional_runtime_substrate | BUILT |
| replayability validation | thunkpunks-constitutional_runtime_substrate | BUILT |
| transformation-to-lineage bridge | thunkpunks-constitutional_runtime_substrate | BUILT |

## Runtime verification update

Local runtime test run:

496 passed in 1.91s

Promoted to BUILT with tests passing:

- runtime GateOutcome closure
- runtime components gate
- coherence checks
- horizon checks
- counterfactual checks
- Lambda0 / SAFE_HOLD pregate layer
- measurement hold layer
- session manager
- event log
- receipts
- lineage
- replay integration
- evidential loop

Current boundary:

- admissibility-kernel: formal/public invariant kernel, 23 tests passing
- constitutional_runtime_substrate: canonical runtime/proof substrate, 496 tests passing
