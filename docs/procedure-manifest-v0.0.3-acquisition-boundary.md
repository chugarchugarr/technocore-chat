# Procedure Manifests v0.0.3 — Acquisition Boundary and Authority Eligibility

**Author:** @chugarchugarr  
**Status:** Draft contribution for upstream review  
**Upstream target:** `formulary-systems/spec`  
**Builds on:** v0.0.2 closure invariant and adjudication state machine

## Problem

v0.0.2 closes the post-observation transformation boundary: given the same committed manifest and the same complete run observations, conforming implementations must derive the same authorized remedy.

The remaining boundary is earlier.

A procedure can still admit the wrong observation while preserving every v0.0.2 transformation rule. In particular:

1. **Selective rerun / equivocation:** a provider or executor can obtain two authentic outputs for the same logical slot and submit the preferred one.
2. **Withholding:** an unfavorable completed result can be suppressed and represented as timeout or transport failure if terminality is not independently verifiable.
3. **Request substitution:** a receipt can bind the right slot to the wrong prompt, evidence root, model profile, or sampling parameters unless the exact execution request is committed.
4. **Namespace regeneration:** a fresh executor-selected `dispute_id` can recreate the authorized slot namespace after results exist.
5. **Scope escalation:** an observation may be authentic and uniquely acquired while establishing less than the contractual consequence requires.

These are all instances of the same rule:

> **Authenticity is necessary for authority, but it is not sufficient for authority.**

## Strengthened closure invariant

> **Every outcome-relevant transformation MUST be committed by the procedure; every observation admitted to those transformations MUST originate from a uniquely authorized execution; and every condition deciding whether that observation is sufficient to carry contractual authority MUST itself be committed and machine-evaluable.**

The v0.0.2 rule remains intact. This proposal inserts acquisition and authority-eligibility semantics before the existing authoritative-result path.

## Normative boundary

```text
evidence
  ↓
authorized execution
  ↓
attested observation
  ↓
authority eligibility
  ↓
authoritative result
  ↓
within-judge aggregation
  ↓
across-panel aggregation
  ↓
requirement resolution
  ↓
contract outcome
  ↓
authorized remedy
  ↓
manifested consequence
```

Only an **eligible** observation may enter the v0.0.2 authoritative-result path.

For an observation `o`:

```text
eligible(o) =
    authorized_execution(o)
  ∧ exact_request_binding(o)
  ∧ unique_terminal_execution(o)
  ∧ sufficient_scope(o)
```

If any term is false or cannot be established under the committed provenance mechanism, the observation MUST NOT acquire authority as a PASS/FAIL result. For v0.0.3, the simplest closed behavior is to map the run to `UNRESOLVED`.

## 1. Preauthorized run slots

Each logical run slot MUST be identifiable before its result exists.

A minimal derivation is:

```text
run_id = H(
  manifest_hash,
  dispute_id,
  requirement_id,
  judge_id,
  run_index
)
```

`run_id` MUST NOT contain the result or any value chosen after observing a result.

`dispute_id` MUST itself be committed or derived from contract/dispute state before any authorized execution can be claimed. An executor MUST NOT be able to choose a fresh `dispute_id` after seeing a result, because that would regenerate the entire slot namespace one level up.

## 2. Submission is not admission

A requester-signed object and a provider-signed object make different claims and MUST remain separate.

```text
submission_commitment
  = requester proves "I sent this request"

claim_receipt
  = provider proves "I accepted this request into this authorized execution slot"
```

A `submission_commitment` MAY be preserved as evidence.

It MUST NOT prove provider admission.

A `claim_receipt` MUST be signed or otherwise attested by the provenance mechanism that has authority to admit the execution.

Silence after a requester submission MUST NOT be interpreted as admission or as `ATTESTED_NO_RESULT` unless the committed provenance mechanism makes that absence independently verifiable.

## 3. Exact request binding

The claim MUST bind the exact execution request, not merely the slot.

A canonical request digest MUST commit every outcome-relevant execution input, including at minimum:

```text
request_hash = H(
  evidence_root,
  prompt_or_material_digest,
  judge_execution_profile,
  model_version_or_weights_pin,
  sampling_parameters,
  transformation/provenance inputs required by the manifest,
  any other manifest-declared outcome-relevant execution input
)
```

The precise encoding belongs in the spec, but the invariant is normative:

> Two executions that can differ in any outcome-relevant input MUST NOT share the same `request_hash`.

## 4. Acquisition lifecycle

The logical run lifecycle is:

```text
AUTHORIZED
    ↓
CLAIMED
    ↓
TERMINAL_RESULT
  | TERMINAL_UNRESOLVED
  | ATTESTED_NO_RESULT
```

### AUTHORIZED

The run slot exists under the committed manifest but no execution mechanism has yet acquired it.

### CLAIMED

The provenance mechanism has accepted exactly one execution attempt for the authorized slot and issued a claim binding at least:

```text
run_id
attempt_id
request_hash
accepted_at
provenance_profile
```

The claim MUST be attributable to the provider/provenance mechanism.

A conforming provenance profile MUST make the claim single-use for the authorized attempt, or otherwise make equivocation detectable and normatively disqualifying.

### TERMINAL_RESULT

The claimed execution produced a conforming terminal observation.

The terminal record MUST bind the claim, terminal status, and output:

```text
terminal_receipt = Attest(
  claim_receipt_hash,
  terminal_status = RESULT,
  output_hash
)
```

### TERMINAL_UNRESOLVED

The claimed execution completed, but its terminal result is explicitly unresolved according to the committed execution/result contract.

This consumes the logical run exactly as `TERMINAL_RESULT` does.

### ATTESTED_NO_RESULT

The provenance mechanism independently establishes that the claimed attempt produced no terminal result under the precommitted failure semantics.

Only this state — or an equivalent independently verifiable absence condition committed by the provenance profile — MAY authorize a retry.

A caller-local timeout, dropped connection, or executor assertion is insufficient.

## 5. Retry semantics

`TERMINAL_RESULT` and `TERMINAL_UNRESOLVED` consume the logical run.

They MUST NOT open another attempt merely because the result is unfavorable.

A retry MAY occur only after `ATTESTED_NO_RESULT` or another manifest-committed, independently verifiable retry condition.

If retries are supported, all permitted attempts MUST be enumerable or derivable before their results exist. For example:

```text
attempt_id = H(run_id, attempt_index)
```

with a manifest-committed maximum attempt count and retry predicate.

Failed attempts remain part of transcript lineage.

## 6. Terminal uniqueness and equivocation

An authentic receipt is not enough if two conflicting authentic receipts can exist for the same claimed attempt.

For every `CLAIMED` attempt, the committed provenance profile MUST provide one of:

1. provider-enforced terminal uniqueness; or
2. independently detectable equivocation with a normative consequence that prevents either conflicting terminal observation from acquiring contractual authority.

A conforming implementation MUST NOT silently choose among multiple valid terminal observations for the same claimed attempt.

If conflicting terminal records exist and the provenance profile does not establish a unique authoritative terminal state, the run is `UNRESOLVED`.

## 7. Authority scope

Provenance answers **where/how this observation came from**.

Authority scope answers **what this observation is sufficient to establish**.

These MUST remain distinct.

Free-form limitations MAY remain evidence-only.

Any limitation capable of altering contractual state MUST enter the authoritative projection as a structured, machine-evaluable field.

The manifest MUST commit:

- the `required_scope` for each requirement or result class; and
- the exact predicate by which an observation scope satisfies that requirement.

For v0.0.3, a minimal rule is:

```text
if observation_scope does not satisfy required_scope:
    run_verdict = UNRESOLVED
```

A VERIFIED/authentic result therefore cannot silently escalate into authority for a consequence outside the scope it actually establishes.

## 8. Proposed schema surface

This is a proposed shape, not a claim that these names are final.

### Manifest

Add an execution-acquisition/provenance section committing:

```json
{
  "acquisition": {
    "dispute_id_rule": "...",
    "run_id_rule": "manifest+dispute+requirement+judge+run_index",
    "provenance_profile": "...",
    "claim_semantics": "...",
    "terminal_semantics": "...",
    "retry_policy": {
      "allowed_after": ["ATTESTED_NO_RESULT"],
      "max_attempts": 1
    }
  }
}
```

Requirement/result configuration should additionally commit `required_scope` and the scope-satisfaction predicate wherever scope can affect authority.

### Acquisition records

A claim record should minimally bind:

```json
{
  "run_id": "...",
  "attempt_id": "...",
  "request_hash": "...",
  "accepted_at": "...",
  "provenance_profile": "...",
  "attestation": "..."
}
```

A terminal record should minimally bind:

```json
{
  "claim_receipt_hash": "...",
  "terminal_status": "RESULT | UNRESOLVED | NO_RESULT",
  "output_hash": "...",
  "attestation": "..."
}
```

`output_hash` is omitted only when the committed terminal status does not carry an output.

## 9. Proposed conformance rules

Following the existing stable C* convention, add new ids rather than rewrite existing rules.

### C16 — preauthorized execution slots

Every LLM-judge run MUST have a deterministic `run_id` derivable from committed pre-result state. `dispute_id` MUST be committed or derived before results exist and MUST NOT be executor-regenerable after observation.

### C17 — exact request binding

Every claimed execution MUST bind a `request_hash` covering every outcome-relevant execution input committed by the manifest.

### C18 — claim authority

A claimed execution MUST carry provider/provenance attestation proving admission into the authorized slot. Requester submission evidence alone MUST NOT satisfy this rule.

### C19 — unique terminal execution

The provenance profile MUST establish a unique terminal state for each claimed attempt, or make equivocation detectable with a committed fail-closed consequence.

### C20 — retry closure

Retry MUST be authorized only by a manifest-committed, independently verifiable terminal absence/failure state. Caller-local timeout or executor assertion MUST NOT authorize another attempt.

### C21 — authority scope

Every scope/limitation capable of changing contractual state MUST be structured and machine-evaluable. The manifest MUST commit the required scope and exact satisfaction predicate.

## 10. Required semantic regression fixtures

This boundary should not merge without executable fixtures for at least these cases.

### F1 — authentic double terminal

Same `run_id`, same claimed attempt, two authentic conflicting terminal outputs.

**Expected:** no implementation may select one silently. If uniqueness cannot be proven, run resolves `UNRESOLVED`.

### F2 — suppressed unfavorable result

A claimed attempt has a real terminal result, but the executor presents a local timeout and requests retry.

**Expected:** retry refused unless the committed provenance mechanism attests `NO_RESULT`.

### F3 — right slot, wrong request

A terminal result is authentic for the authorized `run_id` but the `request_hash` differs in an outcome-relevant input.

**Expected:** observation ineligible; run `UNRESOLVED`.

### F4 — authentic but insufficient scope

A uniquely acquired authentic result carries scope narrower than the manifest's `required_scope`.

**Expected:** observation ineligible; run `UNRESOLVED`.

### F5 — submission without admission

Requester provides a valid `submission_commitment`; no provider claim exists.

**Expected:** attempted submission is preserved as evidence, but execution is not `CLAIMED`, and no result/admission authority follows.

### F6 — namespace regeneration

Executor attempts to create a fresh `dispute_id` after observing an unfavorable terminal result.

**Expected:** refused because the dispute namespace was committed or derived before execution.

## 11. Semantic conformance property for v0.0.3

v0.0.2 currently says:

> Given the same committed manifest and the same complete run observations, two independent conforming implementations MUST derive the same requirement resolutions, the same contract outcome, and the same authorized remedy.

For v0.0.3, strengthen that to:

> **Given the same committed manifest and the same acquisition/transcript evidence, two independent conforming implementations MUST admit the same observations as authoritative and MUST derive the same requirement resolutions, contract outcome, and authorized remedy.**

That moves semantic closure one boundary earlier: not merely deterministic consumption of frozen observations, but deterministic authority over which observations are allowed to enter the consuming state machine.

## 12. Non-claims / layer boundary

This does **not** require every execution provider to support the same provenance mechanism.

It requires the manifest to commit one whose guarantees are sufficient for the authority it is being asked to confer.

If a provider cannot establish single-use acquisition, exact request binding, terminal uniqueness or independently verifiable absence, or required scope, the correct result is refusal or `UNRESOLVED` at that boundary — not an implementation-specific guess.

Likewise, a requester-side submission commitment cannot manufacture provider admission, and cryptographic structure cannot manufacture an authoritative wall-clock or predecessor state that the underlying mechanism does not actually provide.

## 13. Upstream integration target

If this direction survives review, land it as a v0.0.3 spec change touching the full required set together:

- `spec/adjudication.md` — acquisition + eligibility state machine;
- `spec/procedure-manifest.schema.json` — committed acquisition/provenance and required-scope surface;
- `spec/judge-result.schema.json` — structured authority scope if carried by the observation;
- `spec/conformance.md` — new C16+ rules;
- `validator/` — acquisition/eligibility checks;
- `validator/fixtures/` — F1–F6 adversarial regression cases;
- examples + changelog.

This preserves the v0.0.2 result contract and downstream adjudication semantics while closing the authority boundary immediately before them.

## Provenance

This draft formalizes the acquisition boundary developed publicly by @chugarchugarr in the Ethereum Magicians Procedure Manifests RFC thread after the v0.0.1 closure counterexample. The upstream maintainer explicitly proposed v0.0.3 around that boundary and invited @chugarchugarr to land the acquisition boundary or co-draft the state-machine section.

The operative design rule is:

> **Preserve evidence. Re-earn authority.**
