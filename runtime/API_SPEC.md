# API_SPEC

Minimum externally-consumable surface for the Constitutional Runtime Substrate.

This document describes the smallest set of operations an integrator needs to
submit a proposed transformation, receive an admissibility evaluation, obtain
an evidential receipt, and replay the decision deterministically — without
requiring the runtime to surrender any data, decision, or evidence to a
central authority.

The four operations described here correspond to surfaces already present in
the runtime source. Nothing in this document describes capabilities the
runtime does not implement. Fields that an integrator might expect but the
runtime does not currently expose are tagged `NOT_CURRENTLY_EXPOSED`.

## Posture

The runtime is single-process, synchronous, local-by-default Python. This
document describes what an in-process integrator (a calling Python program,
a containerised local service, or an air-gapped batch job) can construct
against the existing module surface. There is **no shipped network API**.
Anything an integrator builds on top of these operations runs in their
deployment, on their hardware, against their data, with their evidence
remaining under their control.

There is **no cloud requirement, no telemetry, no upstream call, no central
corpus, no SDK distribution, and no agent framework.** The runtime evaluates
what it is given and records what it decided. Everything else is the
integrator's choice.

## The four operations

| Name              | Purpose                                                          | Crosses                  |
|-------------------|------------------------------------------------------------------|--------------------------|
| `submit_candidate`  | Construct a typed transformation candidate to be evaluated       | Proposal boundary       |
| `evaluate_candidate`| Decide admissibility for one candidate against current state    | Admissibility boundary  |
| `get_receipt`       | Issue an evidence record binding the decision to the substrate  | Evidence boundary       |
| `replay_decision`   | Reconstruct the evidence chain and verify it has not been altered| Reconstruction boundary |

The boundaries above are not endpoints. They are the only crossings the
runtime currently makes. Other crossings a reader might expect from this
vocabulary — standing, authority assignment, appeal, correction, policy
evolution, legitimacy certification — are **not crossed by this API**.

---

## `submit_candidate`

**Purpose.** Construct a typed transformation candidate that can later be
evaluated. Submission is not evaluation; the candidate is only an input.

**Source.** `runtime/core/types.py::Transformation.new()`
[verified-in-source: runtime/core/types.py:218]

**Request schema.**

```python
Transformation.new(
    expected_delta: dict[str, float] | None = None,
    description: str = "",
    *,
    typed_effects: dict[str, float] | None = None,           # synonym for expected_delta
    intent_class: IntentClass = IntentClass.UNSPECIFIED,
    reversibility: ReversibilityMetadata | None = None,
    expected_commitment_cost: float | None = None,
    provenance_ref: str | None = None,
) -> Transformation
```

Field-by-field:

| Field                       | Tag                    | Notes |
|-----------------------------|------------------------|-------|
| `expected_delta` / `typed_effects` | verified-in-source | Per-coordinate deltas on the field state. Exactly one of the two synonyms must be supplied. |
| `description`               | verified-in-source     | Human-readable rationale. Not used for gating logic. |
| `intent_class`              | verified-in-source     | One of: COMMIT, EXPLORE, STABILIZE, RENEGOTIATE, RECOVER, UNSPECIFIED. Defaults to UNSPECIFIED (the honest default). |
| `reversibility`             | verified-in-source     | `ReversibilityMetadata` carrying class (REVERSIBLE / COSTLY / IRREVERSIBLE / UNKNOWN), reversal cost estimate, rationale. Defaults to UNKNOWN. |
| `expected_commitment_cost`  | verified-in-source     | Estimated τ contribution. Defaults to `max(0.0, typed_effects.get("tau", 0.0))`. |
| `provenance_ref`            | verified-in-source     | Optional link to a `Provenance` record. The runtime treats provenance as a claim, never as truth. |

**Response.** A `Transformation` dataclass with a UUID `transformation_id`.

**Failure modes.**

- `ValueError` if neither `typed_effects` nor `expected_delta` is supplied
  [verified-in-source: runtime/core/types.py:233]
- `ValueError` if both are supplied and differ
  [verified-in-source: runtime/core/types.py:235]

There is no submission-time policy check. The candidate is a structurally
valid input; whether it is admissible is a separate question, answered by
`evaluate_candidate`.

**Reviewability implications.** A submitted candidate is identified by its
`transformation_id` (UUID) and carries its full structural content. The
candidate alone produces no evidence; only evaluation does.

**Replay implications.** None. Submission is a constructor; it touches no
substrate.

**False inference to block.** Submission does not imply standing. The fact
that a caller can construct a `Transformation` does not mean the caller has
authority to act on it, or that the runtime has recognised the caller as a
legitimate proposer. Standing is `NOT_CURRENTLY_EXPOSED`. The runtime
evaluates what it is given; it does not adjudicate who gave it.

---

## `evaluate_candidate`

**Purpose.** Decide admissibility for one candidate against the current field
state under a given threshold set. Returns one of four verdicts from a closed
enumeration. Pure function. Replay-deterministic.

**Source.** `runtime/components/gate.py::evaluate()`
[verified-in-source: runtime/components/gate.py:205]

**Request schema.**

```python
evaluate(gate_input: GateInput) -> GateOutput

GateInput(
    current_state: FieldState,
    transformation: Transformation,
    thresholds: GateThresholds,
    surface_class: str | None = None,
    hbtl_reviewed: bool = False,
    renegotiation_event: bool = False,
    trajectory: Trajectory | None = None,
    horizon_budget: HorizonBudget | None = None,
)
```

Field-by-field:

| Field                  | Tag                  | Notes |
|------------------------|----------------------|-------|
| `current_state`        | verified-in-source   | A `FieldState` with bounded scalars: Ω, ρ, κ, τ, Θ, NSV, plus logical_step. Bounds are enforced at construction. |
| `transformation`       | verified-in-source   | A `Transformation` (as built by `submit_candidate`). |
| `thresholds`           | verified-in-source   | A `GateThresholds` bundle. Parameter-set hash is logged with every outcome via provenance. |
| `surface_class`        | verified-in-source   | Optional surface assignment. Surface Engine not built in v1 [derived-from-types: runtime/components/gate.py:88]. |
| `hbtl_reviewed`        | verified-in-source   | Whether a Human-Before-The-Loop review has been completed for this transformation. |
| `renegotiation_event`  | verified-in-source   | Whether the candidate includes an explicit renegotiation event (suppresses Θ cooling). |
| `trajectory`           | verified-in-source   | Optional trajectory leading to `current_state`. Required for K3 horizon checks. |
| `horizon_budget`       | verified-in-source   | Optional horizon budget. Activates K3 cumulative checks when supplied with a trajectory. |

**Response schema.**

```python
GateOutput(
    outcome: GateOutcome,                       # EXECUTE | TRANSFORM | DEFER | REJECT
    reason_codes: tuple[GateReasonCode, ...],
    predicted_state: FieldState,                # the S(t+1) the gate computed and evaluated
    threshold_crossings: tuple[str, ...],
    rationale: str,                             # human-readable; not used for gating
)
```

[verified-in-source: runtime/components/gate.py:131]

Outcome precedence is **REJECT > DEFER > TRANSFORM > EXECUTE** — the gate
collects all violation signals and selects by explicit precedence, never by
score.

**Failure modes.**

- The gate is a pure function and returns a `GateOutput` for every well-typed
  input. It does not raise on inadmissibility; it returns the verdict.
- `GateInput.__post_init__` raises `ValueError` if a supplied `trajectory`
  does not end at `current_state` [verified-in-source: runtime/components/gate.py:120]
- `apply_delta` enforces bounds and monotonicity invariants on the predicted
  state.

**Reviewability implications.** Every `GateOutput` is fully reconstructable
from its inputs because `evaluate` is a pure function. Given the same
`GateInput`, every reviewer gets the same `GateOutput` — no stochasticity, no
hidden state.

**Replay implications.** Evaluation alone does not record anything. Recording
to the substrate happens when the integrator chooses to issue an event and a
receipt (see `get_receipt`). The gate is the deciding authority; the substrate
is the remembering authority; they are separate by construction.

**False inference to block.** An `EXECUTE` outcome does not imply the
transformation was *legitimate* in any sense beyond what the gate evaluates.
The gate decides admissibility against bounds, coherence (K2), and horizon
(K3) constraints. It does not decide moral, regulatory, jurisdictional, or
institutional legitimacy. Legitimacy is `NOT_CURRENTLY_EXPOSED` and not
claimed.

---

## `get_receipt`

**Purpose.** Bind a gate decision to the evidential substrate as a verifiable
record. The receipt is evidence of what was decided, against which event it
was bound, and within which lineage it sits.

**Source.** `runtime/layers/runtime-layer/receipts/receipts.py::issue_receipt()`
[verified-in-source: runtime/layers/runtime-layer/receipts/receipts.py:136]

**Request schema.**

```python
issue_receipt(
    receipt_id: str,
    transformation_id: str,
    lineage_transformation_id: str,
    event_seq: int,
    event_content_hash: str,
    outcome: str,                                # opaque; receipt does not police
    reason_codes: tuple[str, ...] = (),          # opaque
    evidence_refs: tuple[EvidenceRef, ...] = (),
) -> Receipt
```

The bound event is appended by the integrator to the event log before the
receipt is issued:

```python
EventLog.append(
    session_id: str,
    logical_step: int,
    event_kind: str,
    payload: dict[str, Any],
) -> Receipt   # this is the event-log's own light-weight receipt
```

[verified-in-source: runtime/layers/runtime-layer/event-log/event_log.py:150]

Field-by-field on the receipt itself:

| Field                       | Tag                | Notes |
|-----------------------------|--------------------|-------|
| `receipt_id`                | verified-in-source | Identity of this receipt. Integrator-supplied. |
| `transformation_id`         | verified-in-source | What was evaluated. |
| `lineage_transformation_id` | verified-in-source | Which lineage the transformation belongs to. |
| `event_seq`                 | verified-in-source | The event-log position that recorded the decision. |
| `event_content_hash`        | verified-in-source | Binds the receipt to that event's content. Tampering breaks binding. |
| `outcome`                   | verified-in-source | Opaque string. The receipt records the gate's verdict; it does not validate or police the outcome vocabulary. |
| `reason_codes`              | verified-in-source | Opaque tuple. Recorded as supplied. |
| `evidence_refs`             | verified-in-source | Tuple of `EvidenceRef(kind, ref)` pointing at substrate items supporting the decision. |
| `receipt_hash`              | verified-in-source | Self-hash of the receipt's content. Computed by `Receipt.compute_hash`. |

**Response.** A `Receipt` dataclass with computed `receipt_hash`.

**Verification operations** (read-only):

| Function                        | What it checks |
|---------------------------------|----------------|
| `verify_receipt_self(receipt)`  | The receipt's own content hashes to its stored `receipt_hash`. Detects tampering with outcome, reason codes, or evidence refs. [verified-in-source: receipts.py:171] |
| `verify_receipt_against_log(receipt, event_log)` | Self-hash intact AND bound event exists at `event_seq` with matching `content_hash`. [verified-in-source: receipts.py:185] |
| `verify_receipt_lineage(receipt, lineage_graph)` | Lineage transformation is known to the graph. [verified-in-source: receipts.py:209] |

**Failure modes.**

- `ReceiptViolation` raised by `ReceiptLog.add()` if the receipt's self-hash
  does not verify [verified-in-source: receipts.py:235].
- `verify_receipt_*` return `False` on any binding failure; they do not raise.

**Reviewability implications.** A reviewer with the receipt and the event log
can verify the decision was recorded against the event the receipt claims it
was, and that nothing in the receipt has been altered since issuance. The
verification is mechanical — same inputs, same outputs, no judgement call.

**Replay implications.** Receipts deserialize and re-verify on reload.
Tampered receipts fail self-hash check during `ReceiptLog.add`. The receipt
log is part of the `EvidentialBundle` that `replay_decision` reconstructs.

**False inference to block.** A receipt is **not** an appeal channel. The
receipt records what was decided. It is not a mechanism for contesting the
decision, requesting reconsideration, or escalating to a higher authority.
Appeal is `NOT_CURRENTLY_EXPOSED`. A receipt is also not finality — it is
evidence that a specific decision was made at a specific point; later
decisions about the same transformation in different contexts are independent
events with independent receipts.

---

## `replay_decision`

**Purpose.** Reconstruct the full evidential chain from its serialized form
and verify that nothing has been altered since the chain was last serialized.

**Source.** `runtime/layers/runtime-layer/replay-integration/replay_integration.py::replay_evidential_chain()`
[verified-in-source: runtime/layers/runtime-layer/replay-integration/replay_integration.py:90]

**Request schema.**

```python
replay_evidential_chain(
    bundle: EvidentialBundle,
    *,
    event_log_module: Any,
    lineage_module: Any,
    receipts_module: Any,
) -> ReplayResult

EvidentialBundle(
    event_log_text: str,                       # EventLog.serialize() output
    lineage_records: list[dict[str, Any]],     # serialized LineageRecord dicts
    receipt_log_text: str,                     # ReceiptLog serialized form
)
```

Field-by-field on the bundle:

| Field             | Tag                | Notes |
|-------------------|--------------------|-------|
| `event_log_text`  | verified-in-source | Output of `EventLog.serialize()`. JSON-lines, one event per line. [verified-in-source: event_log.py:233] |
| `lineage_records` | verified-in-source | Serialized `LineageRecord` dicts in append order. |
| `receipt_log_text`| verified-in-source | Serialized `ReceiptLog` text. |

The substrate modules are passed by injection rather than imported, because
the substrate lives under hyphenated directories that are not directly
importable.

**Response schema.**

```python
ReplayResult(
    ok: bool,
    n_events: int,
    n_lineage: int,
    n_receipts: int,
    failures: tuple[str, ...],
)
```

[verified-in-source: replay_integration.py:53]

`ok=True` iff every substrate layer reloaded, reconstructed, and
cross-validated. Any failure is collected with a reason. The function does
not raise on validation failure — it reports — but it does surface the
substrate layers' own fail-closed exceptions as failures in the result.

A fail-closed variant exists:

```python
assert_replayable(bundle, ...) -> ReplayResult
```

Raises `ReplayIntegrityError` on any failure. [verified-in-source: replay_integration.py:182]

**Failure modes.**

- `EventLogViolation` raised by `EventLog.deserialize()` if the loaded chain
  fails integrity verification (tamper, reorder, or corruption detected).
  [verified-in-source: event_log.py:238]
- `LineageViolation` raised by lineage reconstruction on cycles, missing
  parents, or out-of-order ancestry.
- `ReceiptViolation` raised on tampered receipt during reload.
- All surface as failures in `ReplayResult.failures` and trigger
  `ReplayIntegrityError` from `assert_replayable`.

**Reviewability implications.** Replay is the mechanism by which any future
reviewer — auditor, regulator, internal audit, opposing counsel,
post-incident review — can verify that the decision-trace is unchanged from
when it was recorded. The reviewer needs the serialized bundle and the
runtime modules; nothing else.

**Replay implications.** Replay is the inverse of recording. It does not
re-evaluate the gate; it re-verifies the evidence chain. Re-evaluating the
gate is a separate operation: feed the recorded `GateInput` back into
`evaluate` and confirm the same `GateOutput`. The runtime's pure-function
gate guarantees re-evaluation will produce the same verdict.

**False inference to block.** Replay does **not** revise the original
decision. Replay reconstructs and verifies; it does not amend, append, or
correct. If a recorded decision is later judged to have been wrong, the
correction path is forward-only: issue a new decision, with a new event, a
new receipt, and (if the new decision is a recovery) a sanctioned rollback.
Correction is `NOT_CURRENTLY_EXPOSED` as a runtime primitive distinct from
forward evidence; rollback is built [verified-in-source: runtime/components/rollback.py]
as the only sanctioned τ-decrement, NSV-honest.

---

## Ingress (for completeness)

External observations enter the substrate through the ingress membrane before
any of the above is reached. The membrane is **not** part of this minimum API
because it does not produce decisions — it refuses or admits observations
as state. But integrators handling external data should route it through
the membrane before using it to construct a `FieldState`.

Source: `runtime/layers/ingress-layer/ingress/ingress_membrane.py::admit()` /
`try_admit()` [verified-in-source: ingress_membrane.py:142]

The membrane refuses, fail-closed, on:

- Authority field at any nesting depth (`RefusalReason.AUTHORITY_FIELD`)
- Verdict vocabulary anywhere in the payload (`RefusalReason.VERDICT_VALUE`)
- Fabricated confidence outside `(0.0, 1.0]` on populated coordinates
- Dishonest coordinate (populated without value, unpopulated with value)
- Silent compression (lossy coordinate without disclosure)
- Malformed observation

The membrane records provenance as a *claim*, not as truth — the
`Provenance.asserted_true` field is structurally always `False`. Memory is
not truth; the membrane never asserts otherwise.

---

## Deployment modes

The four operations above are pure in-process Python. They can run in any of:

| Mode               | Notes |
|--------------------|-------|
| **Local**          | Python process on a developer machine. The simplest case; default for evaluation and review. |
| **On-premises**    | Python process on customer hardware, within the customer's network. No external dependency. |
| **Private VPC**    | Python process in a customer-controlled cloud VPC. No outbound calls from the runtime; any network surface is the integrator's. |
| **Air-gapped**     | Python process with no network connectivity. The runtime requires no network access to evaluate, record, or replay. Replay across machines uses serialized `EvidentialBundle` artifacts copied over an isolated channel. |
| **Classified / sovereign** | Same as air-gapped, with additional integrator-applied controls. The runtime's behaviour does not change; the integrator's environment determines the security posture. |

In every mode, **the integrator's data, state, decisions, and evidence remain
under the integrator's control.** The runtime has no upstream destination.

---

## Security assumptions

- The runtime trusts its own process boundary. An attacker with arbitrary
  code execution in the runtime's Python interpreter can do anything they
  want; this is true of every in-process library.
- Integrity of the evidence chain depends on the cryptographic strength of
  SHA-256 (used for content and chain hashes). The runtime does not address
  threat models that assume SHA-256 collisions.
- The runtime makes no claim about its host environment. The integrator
  supplies the operational security perimeter: process isolation, file
  permissions, key management, network controls.
- The runtime makes no claim about authenticity of inputs. An attacker who
  controls the integrator's input pipeline can submit any `FieldState` and
  any `Transformation`. The runtime evaluates what it receives; it does not
  verify upstream identity. Identity is the integrator's concern.

---

## Non-claims

The following are **not** what this API does, regardless of what the
vocabulary might suggest:

- **No SDK.** This document describes existing module surfaces, not a
  packaged client library. There is no `pip install neverthought-client`.
- **No cloud API.** There is no hosted endpoint. There is no service. There
  is no `POST /v1/evaluate`. The runtime is in-process Python.
- **No corpus.** The runtime accumulates evidence locally in the integrator's
  deployment. There is no central corpus, no shared registry, no telemetry
  upload, no analytics surface.
- **No standing.** The runtime does not adjudicate who is permitted to
  submit. Submission is open at the API surface; legitimacy of the submitter
  is the integrator's policy concern, not the runtime's.
- **No appeal.** The runtime does not provide a mechanism for contesting,
  overturning, or escalating a decision. Appeal is an institutional concept
  layered above the runtime by the integrator.
- **No correction primitive.** Errors are corrected forward by new
  decisions, new events, new receipts, and (for recovery) sanctioned
  rollbacks. There is no in-place edit.
- **No policy evolution.** The constitution (K1/K2/K3 invariants) is held
  fixed at runtime. Changing it is a build-time act outside the runtime's
  scope. There is no `evolve_policy()`.
- **No legitimacy certification.** An `EXECUTE` outcome is admissibility
  evidence, not a certificate of legitimacy. Legitimacy in any moral,
  regulatory, or institutional sense is outside this API.
- **No agent framework.** The runtime is a substrate; it does not act, plan,
  or pursue goals. It evaluates what it is given.
- **No production deployment claim.** The runtime is a research artifact.
  It is not deployed and carries no operational decisions.

---

## Reviewer walkthrough

A reviewer should be able, in a single sitting, to:

1. **Submit a candidate.** Construct a `Transformation` with `Transformation.new(...)`.
2. **Evaluate it.** Pass a `GateInput` (state + candidate + thresholds) to
   `gate.evaluate()`. Confirm the returned `GateOutput` has one of four
   outcomes and a tuple of reason codes.
3. **Record an event.** Append to an `EventLog` via `.append(...)`. Receive
   an event-log receipt (lightweight; carries `seq`, `content_hash`, `chain_hash`).
4. **Issue a receipt.** Call `issue_receipt(...)` with the event seq, content
   hash, lineage id, outcome string, and reason codes. Receive a `Receipt`
   with a self-hash.
5. **Verify the receipt.** Call `verify_receipt_self`, `verify_receipt_against_log`,
   and `verify_receipt_lineage`. Confirm each returns `True`.
6. **Tamper with the receipt.** Modify any field of the `Receipt` dataclass
   (by reconstructing it with altered content). Confirm
   `verify_receipt_self` now returns `False`.
7. **Serialize the chain.** Get the `EventLog.serialize()` text, the lineage
   records, and the `ReceiptLog` serialized form. Bundle into an
   `EvidentialBundle`.
8. **Replay.** Call `replay_evidential_chain(bundle, ...)`. Confirm
   `ReplayResult.ok == True`.
9. **Tamper with the bundle.** Edit one byte of `event_log_text`. Replay
   again. Confirm `ok == False` and a failure is reported.
10. **Read** `runtime/NON_CLAIMS.md` and `runtime/PROOF_SURFACES.md`. Confirm
    the non-claims align with what this document also refuses to claim, and
    that the proof surfaces are runnable.

Every step above is reproducible against the source tree at the commit this
spec was verified against. If any step fails, this document is wrong against
its source and should be corrected before being trusted.

---

## What remains under integrator control

- The data (state, transformation candidates, observations, evidence).
- The decision (the gate runs locally; verdicts never leave the integrator's process).
- The audit trail (event log, lineage graph, receipt log all live in
  integrator-controlled storage).
- The deployment posture (on-prem, VPC, air-gapped, sovereign — runtime
  behaviour does not change).
- The integration surface (how `submit_candidate` is exposed to the
  integrator's users is the integrator's decision; the runtime ships no
  endpoint).

The runtime is the substrate. Everything around it — identity, authority,
appeal, policy evolution, certification, productisation — is the
integrator's concern, and the runtime does not pretend to address any of it.

---

## Source verification

This document was generated against the source tree extracted from
`thunkpunks-constitutional_runtime_substrate-main.zip`.

Verification gates passed at generation time:

- `python3 -m pytest runtime/tests -q` → **496 passed**
- `python3 runtime/tests/test_closed_circuit_fixture.py` → **replay: VERIFIED**

Every `[verified-in-source: path]` tag in this document refers to a file and
location verified to exist and to contain the described surface at the time
of generation. The file `source_mapping_notes.md` enumerates each
field-to-source binding.
