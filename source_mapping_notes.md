# source_mapping_notes.md

Per-field source verification for `runtime/API_SPEC.md`.

Each row maps a documented field or operation to the exact location in the
verified source tree. Tags used:

- `verified-in-source` — field appears in the source at the cited path; signature, type, and behaviour match what the spec describes.
- `derived-from-types` — field is documented from a type definition / docstring rather than a public function signature.
- `NOT_CURRENTLY_EXPOSED` — concept mentioned in the spec as a non-claim; no corresponding surface in the source.

Source root: extracted `thunkpunks-constitutional_runtime_substrate-main.zip`
into `/home/claude/thunkpunks-constitutional_runtime_substrate-main/`.
Verification gates: `pytest runtime/tests -q` → 496 passed;
`python3 runtime/tests/test_closed_circuit_fixture.py` → VERIFIED.

---

## submit_candidate

| Documented item             | Tag                  | Source path                                       | Line  |
|-----------------------------|----------------------|----------------------------------------------------|-------|
| `Transformation.new(...)`   | verified-in-source   | runtime/core/types.py                              | 218   |
| `transformation_id` (UUID)  | verified-in-source   | runtime/core/types.py                              | 246   |
| `typed_effects`             | verified-in-source   | runtime/core/types.py                              | 197   |
| `expected_delta` (synonym)  | verified-in-source   | runtime/core/types.py                              | 209   |
| `intent_class`              | verified-in-source   | runtime/core/types.py                              | 119–135 |
| `IntentClass` enum values   | verified-in-source   | runtime/core/types.py                              | 129–135 |
| `reversibility`             | verified-in-source   | runtime/core/types.py                              | 152–168 |
| `ReversibilityClass` values | verified-in-source   | runtime/core/types.py                              | 138–149 |
| `expected_commitment_cost`  | verified-in-source   | runtime/core/types.py                              | 200   |
| `provenance_ref`            | verified-in-source   | runtime/core/types.py                              | 201   |
| `description`               | verified-in-source   | runtime/core/types.py                              | 202   |
| ValueError on missing input | verified-in-source   | runtime/core/types.py                              | 233   |
| ValueError on conflict      | verified-in-source   | runtime/core/types.py                              | 235   |

---

## evaluate_candidate

| Documented item             | Tag                  | Source path                                       | Line  |
|-----------------------------|----------------------|----------------------------------------------------|-------|
| `evaluate(gate_input)`      | verified-in-source   | runtime/components/gate.py                         | 205   |
| `GateInput` dataclass       | verified-in-source   | runtime/components/gate.py                         | 79    |
| `GateThresholds` dataclass  | verified-in-source   | runtime/components/gate.py                         | 52    |
| `GateOutput` dataclass      | verified-in-source   | runtime/components/gate.py                         | 130   |
| `GateOutcome` enum (closed) | verified-in-source   | runtime/core/types.py                              | 64–74 |
| Outcome precedence          | verified-in-source   | runtime/components/gate.py                         | 211–223 (docstring) |
| `current_state: FieldState` | verified-in-source   | runtime/components/gate.py / runtime/core/types.py | 82 / 27 |
| `transformation`            | verified-in-source   | runtime/components/gate.py                         | 83    |
| `thresholds`                | verified-in-source   | runtime/components/gate.py                         | 84    |
| `surface_class` (optional)  | verified-in-source   | runtime/components/gate.py                         | 88    |
| `hbtl_reviewed`             | verified-in-source   | runtime/components/gate.py                         | 92    |
| `renegotiation_event`       | verified-in-source   | runtime/components/gate.py                         | 95    |
| `trajectory` (optional)     | verified-in-source   | runtime/components/gate.py                         | 107   |
| `horizon_budget` (optional) | verified-in-source   | runtime/components/gate.py                         | 113   |
| Trajectory-coherence check  | verified-in-source   | runtime/components/gate.py                         | 115–127 |
| `outcome` field             | verified-in-source   | runtime/components/gate.py                         | 140   |
| `reason_codes` tuple        | verified-in-source   | runtime/components/gate.py                         | 141   |
| `predicted_state` FieldState| verified-in-source   | runtime/components/gate.py                         | 142   |
| `threshold_crossings`       | verified-in-source   | runtime/components/gate.py                         | 143   |
| `rationale` (non-policy)    | verified-in-source   | runtime/components/gate.py                         | 144   |
| `GateReasonCode` enum       | verified-in-source   | runtime/core/types.py                              | 76–113 |
| Pure-function property      | verified-in-source   | runtime/components/gate.py                         | 206 (docstring "Pure function. Replay-deterministic.") |

---

## get_receipt

| Documented item                       | Tag                  | Source path                                                        | Line |
|---------------------------------------|----------------------|---------------------------------------------------------------------|------|
| `issue_receipt(...)`                  | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 136  |
| `Receipt` dataclass                   | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 86   |
| `Receipt.compute_hash`                | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 119  |
| `outcome` (opaque)                    | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 95–98 (docstring) |
| `reason_codes` (opaque)               | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 100  |
| `evidence_refs` (`EvidenceRef`)       | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 68–81 |
| `event_seq` binding                   | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 99   |
| `event_content_hash` binding          | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 99   |
| `lineage_transformation_id` binding   | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 97   |
| `receipt_hash` (self-hash)            | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 102  |
| Receipt does not police outcomes      | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 92–95 (docstring) |
| `verify_receipt_self`                 | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 171  |
| `verify_receipt_against_log`          | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 185  |
| `verify_receipt_lineage`              | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 209  |
| `ReceiptLog.add()` self-hash check    | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 233–238 |
| `ReceiptViolation` raised on tamper   | verified-in-source   | runtime/layers/runtime-layer/receipts/receipts.py                  | 56   |
| `EventLog.append(...)`                | verified-in-source   | runtime/layers/runtime-layer/event-log/event_log.py                | 150  |
| Event-log Receipt dataclass           | verified-in-source   | runtime/layers/runtime-layer/event-log/event_log.py                | 108  |
| `content_hash` binding semantics      | verified-in-source   | runtime/layers/runtime-layer/event-log/event_log.py                | 92–98 |

---

## replay_decision

| Documented item                       | Tag                  | Source path                                                                       | Line |
|---------------------------------------|----------------------|------------------------------------------------------------------------------------|------|
| `replay_evidential_chain(...)`        | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 90   |
| `assert_replayable(...)`              | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 182  |
| `EvidentialBundle` dataclass          | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 71   |
| `event_log_text`                      | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 78   |
| `lineage_records`                     | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 79   |
| `receipt_log_text`                    | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 80   |
| `ReplayResult` dataclass              | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 53   |
| `ok / n_events / n_lineage / n_receipts / failures` | verified-in-source | runtime/layers/runtime-layer/replay-integration/replay_integration.py | 54–58 |
| `EventLog.serialize()`                | verified-in-source   | runtime/layers/runtime-layer/event-log/event_log.py                                | 233  |
| `EventLog.deserialize()` fail-closed  | verified-in-source   | runtime/layers/runtime-layer/event-log/event_log.py                                | 238  |
| `EventLog.verify_integrity()`         | verified-in-source   | runtime/layers/runtime-layer/event-log/event_log.py                                | 192  |
| `ReplayIntegrityError`                | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 48   |
| Dependency-injection rationale        | verified-in-source   | runtime/layers/runtime-layer/replay-integration/replay_integration.py             | 96–101 (docstring) |

---

## Lineage (referenced from get_receipt / replay_decision)

| Documented item             | Tag                | Source path                                                | Line |
|-----------------------------|--------------------|-------------------------------------------------------------|------|
| `LineageRecord` dataclass   | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py             | 48   |
| `LineageGraph`              | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py             | 79   |
| `parent_ids` (from composed_from) | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py        | 62   |
| `LineageGraph.add()`        | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py             | 98   |
| `LineageGraph.ancestry()`   | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py             | 145  |
| `LineageGraph.validate_replayable()` | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py    | 173  |
| `LineageViolation` raised on cycle/parent missing | verified-in-source | runtime/layers/runtime-layer/lineage/lineage.py | 44 |

---

## Ingress membrane (referenced for completeness)

| Documented item                    | Tag                | Source path                                              | Line |
|------------------------------------|--------------------|-----------------------------------------------------------|------|
| `admit(raw) -> AdmittedObservation`| verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 142  |
| `try_admit(raw) -> (admitted, reason)` | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 234 |
| `AdmissionRefusal` exception       | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 56   |
| `RefusalReason` enum               | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 60–66 |
| `Provenance.asserted_true=False`   | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 70–82 |
| `AdmittedObservation` schema       | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 89   |
| Recursive authority-field scan     | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 110  |
| Recursive verdict-value scan       | verified-in-source | runtime/layers/ingress-layer/ingress/ingress_membrane.py | 131  |

---

## FieldState (used by evaluate_candidate)

| Documented item              | Tag                | Source path              | Line |
|------------------------------|--------------------|---------------------------|------|
| `FieldState` dataclass       | verified-in-source | runtime/core/types.py     | 27   |
| Bounded scalars: Ω, ρ, κ, τ, Θ, NSV, logical_step | verified-in-source | runtime/core/types.py | 36–41 |
| Bounds enforcement in `__post_init__` | verified-in-source | runtime/core/types.py | 43–55 |
| τ, NSV monotone non-decreasing | derived-from-types | runtime/core/types.py     | 5–6 (module docstring) |
| Frozen (immutable snapshots) | verified-in-source | runtime/core/types.py     | 26   |

---

## Recovery / rollback (referenced for completeness)

| Documented item                 | Tag                  | Source path                          | Notes |
|---------------------------------|----------------------|---------------------------------------|-------|
| Rollback is sole sanctioned τ-decrement | verified-in-source | runtime/components/rollback.py | Confirmed present as module |
| Correction-is-forward statement | derived-from-types   | runtime/components/rollback.py + REVIEWER_GUIDE.md | Forward-only design |

---

## NOT_CURRENTLY_EXPOSED (named in spec as non-claims)

The following are referenced in API_SPEC.md as concepts the runtime does
**not** implement. Each is named to block a likely reader-side false inference.
No source path exists for any of these.

| Concept                              | Tag                    | Reason |
|--------------------------------------|------------------------|--------|
| Standing predicate                   | NOT_CURRENTLY_EXPOSED  | Runtime has no per-actor standing check; gate is sole authority and is structurally bounded, but no standing schema. Per prior audit: candidate for `standing_as_recorded_evidence` only, not promoted. |
| Authority assignment as a primitive  | NOT_CURRENTLY_EXPOSED  | Authority is bounded structurally (AUTHORITY_MAP, no shadow kernel), not as a per-decision predicate. |
| Appeal                               | NOT_CURRENTLY_EXPOSED  | No mechanism for contesting / re-opening a decision. |
| Correction (in-place)                | NOT_CURRENTLY_EXPOSED  | Correction is forward-only via new events + receipts; rollback is the only sanctioned τ-decrement. No in-place edit. |
| Policy evolution                     | NOT_CURRENTLY_EXPOSED  | Constitution (K1/K2/K3) is held fixed at runtime. Changing it is a build-time act. |
| Legitimacy certification             | NOT_CURRENTLY_EXPOSED  | Gate decides admissibility, not legitimacy in any moral/regulatory sense. |
| SDK / client library                 | NOT_CURRENTLY_EXPOSED  | No packaged distribution. |
| Cloud API / hosted endpoint          | NOT_CURRENTLY_EXPOSED  | In-process Python only. |
| Central corpus / telemetry           | NOT_CURRENTLY_EXPOSED  | No upstream destination. |
| Agent framework                      | NOT_CURRENTLY_EXPOSED  | Substrate, not agent. |

---

## Collapse-detector pass (orientation constraint applied)

API_SPEC.md was checked against the seven boundary-hygiene tests from the
orientation constraint. For each verb, a likely reader-side false inference
was explicitly named and blocked in the spec text:

| Verb               | False inference blocked                            | Spec section |
|--------------------|----------------------------------------------------|--------------|
| submit_candidate   | submission implies standing                        | "False inference to block" under submit_candidate |
| evaluate_candidate | EXECUTE implies legitimate transformation          | "False inference to block" under evaluate_candidate |
| get_receipt        | receipt implies appeal or finality                 | "False inference to block" under get_receipt |
| replay_decision    | replay implies revision                            | "False inference to block" under replay_decision |

Boundary-hygiene checks performed:

- score ≠ standing: spec contains no scoring language; gate verdicts are categorical (closed enum); standing is named NOT_CURRENTLY_EXPOSED.
- evidence ≠ authority: receipts are described as evidence binding outcome to event; receipt module explicitly does not police outcomes (verified in source docstring).
- policy ≠ legitimacy: spec uses "admissibility" for what the gate decides; "legitimacy" is named only in non-claims.
- execution ≠ legitimate transformation: EXECUTE is described as admissibility evidence, not legitimacy certification.
- receipt ≠ appeal: explicit non-claim "no appeal" + "false inference to block" under get_receipt.
- replay ≠ correction: explicit non-claim "no correction primitive" + "false inference to block" under replay_decision.
- recorded ≠ built: every spec field is tagged verified-in-source / derived-from-types / NOT_CURRENTLY_EXPOSED; orientation-only items do not appear in built fields.

Six prohibited endpoints from the constraint were checked-and-absent from the spec:
- submit_appeal — absent ✓
- record_standing — absent ✓
- evolve_policy — absent ✓
- correct_decision — absent ✓
- certify_legitimacy — absent ✓
- export_corpus — absent ✓
