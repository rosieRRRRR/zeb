# ZEB -- Zero Exposure Broadcast

**An Open Standard for Execution-Gated Transactions and Bitcoin Broadcast Discipline**

* **Specification Version:** 1.3.0
* **Status:** Implementation Ready
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea
* **Scope:** Bitcoin current consensus and policy. No new opcodes. No consensus changes.
* **PQ Ecosystem:** CORE — The PQ Ecosystem is a post-quantum security framework built on deterministic enforcement, fail-closed semantics, and refusal-driven authority. Bitcoin is the reference deployment. It is not the scope.


This repository contains two specifications:

ZET defines a rail-agnostic execution boundary that separates intent from execution and enforces atomic execute-or-refuse semantics after external enforcement approval.

ZEB defines the Bitcoin execution profile that implements the ZET boundary, specifying broadcast, observation, exposure detection, and confirmation tracking for Bitcoin transactions.

ZET is not Bitcoin-specific. ZEB is Bitcoin-specific.

---

## Summary

ZEB defines a broadcast boundary that separates authorisation from transaction broadcast.

It ensures that broadcast occurs only after external approval and provides observation and exposure-detection mechanisms for transaction workflows.

ZEB does not decide whether an action is allowed. Broadcast proceeds only after PQSEC has produced an approval outcome.

---

## ZET Packaging Statement (Normative)

ZEB is the primary execution boundary specification.

ZET (Zero-Exposure Transaction) concepts are treated as Part I of ZEB and do not form a separate authority surface. ZET patterns describe execution techniques; ZEB defines the boundary discipline.

**Implications:**

1. ZET is not a standalone specification
2. ZET patterns are execution mechanics, not authority sources
3. All authority decisions originate in PQSEC
4. ZEB implements the Bitcoin-specific execution profile

---

## Conformance Keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## Explicit Dependencies

| Specification | Minimum Version | Purpose |
|---------------|-----------------|---------|
| PQSEC | ≥ 2.0.3 | EnforcementOutcome production and consumption |
| Epoch Clock | ≥ 2.1.0 | Tick-based deadline enforcement |
| PQSF | ≥ 2.0.3 | Canonical encoding (when session binding is used) |

ZET is rail-agnostic and has no Bitcoin-specific dependencies.

Hashing note: ZEB defines no independent hashing constructions. Any hash values used or consumed by ZEB (including `intent_hash`-bound execution artefacts) follow PQSF §9 Tier 2 (SHAKE256-256, 32 bytes) unless a Bitcoin Script context requires SHA-256.

---

## Part I -- ZET: Zero-Exposure Transactions (Execution Boundary)

### Summary

ZET defines a strict execution boundary that separates intent formation from execution.

Intents are non-authoritative declarations that are safe to observe and analyze and carry zero execution capability.

Execution occurs only after external enforcement approval from PQSEC and is atomic: execute or refuse, with no partial states.

ZET prevents pre-construction and execution-token attacks by ensuring no executable artefacts exist before enforcement completes.

ZET provides execution mechanics only. All authority decisions occur in PQSEC.

---

### Scope and Execution Boundary

ZET defines execution mechanics only.

ZET defines:
- strict phase separation between intent, evaluation, and execution
- a single atomic execution boundary
- consumption of external enforcement outcomes
- replay-safe execution outcome binding

ZET does not define custody authority, predicate evaluation, time issuance, broadcast mechanics, settlement semantics, or cryptographic primitives. Runtime attestation is handled exclusively by PQSEC.

---

### Architecture Overview

ZET defines a three-phase architecture:

1. Intent formation  
2. External evaluation  
3. Atomic execution  

No artefact produced in earlier phases carries execution authority.

---

### Authority Boundary

ZET grants no authority.

ZET executes or refuses solely based on externally supplied enforcement outcomes from PQSEC.

---

### Replay Protection Requirement

ZET MUST enforce replay protection on EnforcementOutcome artefacts.

Each decision_id MUST be accepted at most once.

Replays MUST result in refusal with error code `E_OUTCOME_REPLAYED`.

---

### Time Semantics

All time bounds in ZET MUST be expressed in Epoch Clock ticks.

Local system clocks MUST NOT be used for execution deadlines or authority decisions.

---

### ZET Execution Integration

When executing Bitcoin transactions, ZET delegates execution to ZEB.

ZET remains rail-agnostic. Bitcoin execution is a profile, not the core.

---

## Part II -- ZEB: Zero-Exposure Execution Boundary (Bitcoin Profile)

### Summary

ZEB provides Bitcoin-specific broadcast discipline and exposure detection.

ZEB gates broadcast on a valid PQSEC EnforcementOutcome, monitors observation sources for deterministic exposure detection, and tracks confirmation status.

ZEB provides zero security benefit without execution-gated spend construction.  
Zero-exposure properties arise only from the combination of:
- ZET execution boundary
- ZEB broadcast and observation
- SEAL encrypted submission

ZEB provides broadcast mechanics and observation only.  
No authority. No spend construction. No enforcement.

---

### 1A. Canonical Scope and Limitation Disclaimer (Normative)

ZEB does not claim censorship resistance, miner-inclusion guarantees, replacement-proof execution, or post-broadcast quantum immunity.

ZEB provides broadcast discipline and observation mechanics only. It does not construct spends, does not grant authority, and does not make policy or custody decisions. ZEB executes solely after receipt and verification of a valid EnforcementOutcome produced by PQSEC and only through the ZET execution boundary.

Any zero-exposure or reduced-exposure property is conditional on correct composition with:

* SEAL encrypted submission,
* strict attempt-scoped burn discipline, and
* correct enforcement-to-broadcast sequencing.

These properties are not standalone guarantees and MUST NOT be represented as such.

---

### Scope and Broadcast Boundary

ZEB defines transaction broadcast and observation mechanics only.

ZEB defines:
- submission modes
- exposure detection
- confirmation tracking
- deterministic failure signaling

ZEB implements the ZET execution boundary interface for Bitcoin.

ZEB introduces no new opcodes, no consensus changes, and no miner coordination.

---

### Execution-Gated Spend Dependency

ZEB MUST be used only with execution-gated spend constructions when claiming zero-exposure or replacement-resistance properties.

ZEB provides zero security benefit without execution-gated spend construction.

ZEB MUST NOT be represented as replacement-proof or censorship-resistant.

---

### Burn Requirement

If a ZEB execution attempt fails due to:
- exposure detection
- confirmation timeout
- enforcement invalidation

then all attempt-scoped execution material, including S1, MUST be considered permanently burned.

Wallets MUST mark the associated intent_hash and secrets as unusable.

Burn does not imply permanent loss of funds. Alternative spending paths using new intents remain possible.

---

### Threat Model Summary

ZEB addresses public-mempool reaction attacks whose feasibility depends on constructing a valid competing spend during the broadcast-to-confirmation window.

ZEB does not protect against pre-compromised keys, miner-colluding adversaries, or censorship.

---

## Error Codes (Normative)

- `E_OUTCOME_MISSING` -- No EnforcementOutcome provided  
- `E_OUTCOME_REPLAYED` -- decision_id already used  
- `E_DENIED` -- PQSEC refused authorization  
- `E_BURNED_INTENT` -- intent_hash marked as burned  
- `E_EXPOSURE_DETECTED` -- transaction observed by quorum  
- `E_CONFIRMATION_TIMEOUT` -- deadline exceeded before confirmation  

---

## Execution Recovery Semantics (Normative)

When a ZEB execution attempt reaches a terminal state (any failure, exposure detection, confirmation timeout, or enforcement invalidation), the execution attempt is permanently ended. No automatic retry, implicit resumption, or unattended recovery is permitted.

"Explicit authorisation required for recovery" means: a fresh PQSEC ALLOW decision for either a new intent (new intent_hash, new decision_id) or a recovery-class operation evaluated under the same enforcement rules as any other Authoritative operation. The original EnforcementOutcome and decision_id MUST NOT be reused. The original intent_hash is burned and MUST NOT be resubmitted.

Implementations MUST NOT implement automatic retry logic, timer-based re-execution, or silent fallback to alternative broadcast methods. Recovery is a deliberate human or policy decision surfaced through the standard BPC/PQSEC pipeline.

---

## Observer Quorum Guidance (Informative)

Observer quorum configuration:
- public mode: quorum = ∞ (never triggers exposure)
- restricted or hybrid: policy-defined (for example, 3)
- default: 1 (conservative)

Observer quorum refers to independent observation sources and is distinct from PQSEC predicate quorums.

---

## Network Partition Handling (Informative)

During network partitions:
- ZEB may fail to broadcast or observe transactions
- deadlines continue to be enforced using Epoch Clock ticks
- burn discipline still applies
- recovery occurs via a new intent after partition resolution

---

## Conformance Targets

1. ZET Conformant  
2. ZEB Conformant  
3. Bitcoin Zero-Exposure Execution Conformant (ZET + ZEB + SEAL + PQSEC)

---

## Annex A -- ZET Execution Boundary Interface (Normative)

## A.1 EnforcementOutcome

**Note (Normative):** PQSEC §15.1 is the authoritative definition of EnforcementOutcome. This Annex A.1 dataclass is illustrative only and MUST NOT be treated as a normative schema. Implementations MUST parse and verify EnforcementOutcome using PQSEC rules and canonical bytes.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class EnforcementOutcome:
    decision: str          # "ALLOW" / "DENY" / "FAIL_CLOSED_LOCKED"
    decision_id: str
    intent_hash: bytes
    session_id: bytes
    exporter_hash: bytes
    issued_tick: int
    expiry_tick: int
```

---

## A.2 ExecutionResult

```python
from dataclasses import dataclass
from typing import Optional, Dict, Any

@dataclass(frozen=True)
class ExecutionResult:
    status: str
    execution_id: str
    error_code: Optional[str] = None
    rail_result: Optional[Dict[str, Any]] = None
```

---

## A.3 ZET Boundary Contract with Replay Guard

```python
_seen_decisions = set()

def zet_execute(intent: dict, outcome: EnforcementOutcome, rail_executor) -> ExecutionResult:
    if outcome is None:
        return ExecutionResult(
            status="REFUSED",
            execution_id="none",
            error_code="E_OUTCOME_MISSING"
        )

    if outcome.decision_id in _seen_decisions:
        return ExecutionResult(
            status="REFUSED",
            execution_id=outcome.decision_id,
            error_code="E_OUTCOME_REPLAYED"
        )

    if outcome.decision != "ALLOW":
        _seen_decisions.add(outcome.decision_id)
        return ExecutionResult(
            status="REFUSED",
            execution_id=outcome.decision_id,
            error_code="E_DENIED"
        )

    _seen_decisions.add(outcome.decision_id)
    return rail_executor.execute(intent, outcome)
```

### A.3A Replay Guard Durability (Normative)

Implementations MUST ensure the `_seen_decisions` store is:

1. persistent across restarts,
2. thread-safe (or otherwise concurrency-safe), and
3. scoped to the local signing/execution domain.

If `_seen_decisions` state is lost, corrupted, or cannot be proven intact, the implementation MUST fail closed for Authoritative execution by returning `E_OUTCOME_REPLAYED` (or `E_FAIL_CLOSED_LOCKED` where the policy requires lockout) until replay state is recovered (for example by restoring from a trusted backup).

Replay records MAY be pruned after the corresponding EnforcementOutcome `expiry_tick` has passed.

A `decision_id` is considered consumed once `_seen_decisions` is updated, regardless of whether the rail executor succeeds or fails. Retries MUST require a new EnforcementOutcome with a new `decision_id`.

---

## Annex B -- ZEB Bitcoin Rail Executor (Normative)

## B.1 Executor Interface

```python
class ZEBExecutor:
    def execute(self, intent: dict, outcome: EnforcementOutcome) -> ExecutionResult:
        raise NotImplementedError
```

---

## B.2 Attempt Identity (Normative)

```python
import uuid
from dataclasses import dataclass

@dataclass(frozen=True)
class AttemptIdentity:
    attempt_id: str
    intent_hash: bytes
    created_tick: int

# Note: attempt_id need not be cryptographically random, but MUST be unique
# across all attempts in the deployment scope.
def new_attempt(intent_hash: bytes, tick: int) -> AttemptIdentity:
    return AttemptIdentity(
        attempt_id=str(uuid.uuid4()),
        intent_hash=intent_hash,
        created_tick=tick
    )
```

---

## Annex C -- ZEB Submission Plane (Reference)

```python
class Submitter:
    def submit_public(self, rawtx_hex: str) -> str:
        raise NotImplementedError

    def submit_restricted(self, rawtx_hex: str, endpoints: list[str]) -> str:
        raise NotImplementedError
```

**SEAL Integration:** When SEAL is deployed, ZEB's `submit_restricted()` delegates to SEAL's encrypted submission protocol. The `rawtx_hex` parameter is encrypted by SEAL before transmission to the Submission Endpoint. ZEB's submission plane is an abstraction; SEAL provides the concrete restricted-mode implementation. ZEB's `submit_public()` bypasses SEAL entirely (standard Bitcoin broadcast).

---

## Annex D -- Observation Plane (Normative)

```python
from typing import Optional

class Observer:
    def seen_in_public_mempool(self, txid: str) -> bool:
        raise NotImplementedError

    def confirmed_height(self, txid: str) -> Optional[int]:
        raise NotImplementedError
```

---

## Annex E -- Exposure Detection (Normative)

```python
def exposure_detected(mode: str, mempool_hits: int, observer_quorum: int) -> bool:
    if mode == "public":
        return False
    return mempool_hits >= observer_quorum
```

---

## Annex F -- Tick-Based Deadline Enforcement (Normative)

`deadline_tick` MUST be less than or equal to `outcome.expiry_tick`.

Policy MAY set `deadline_tick` lower than `outcome.expiry_tick`, but MUST NOT set it higher.

```python
def deadline_exceeded(current_tick: int, deadline_tick: int) -> bool:
    return current_tick >= deadline_tick
```

---

## Annex G -- Burn Discipline (Normative)

```python
burned_intents = set()

def burn_attempt(intent_hash: bytes):
    burned_intents.add(intent_hash)

def is_burned(intent_hash: bytes) -> bool:
    return intent_hash in burned_intents
```

Implementations MUST ensure the burned_intents store is persistent and thread-safe.

---

## Annex H -- Full ZEB Execution Loop with S1 Revelation (Reference)

```python
def submit_transaction(rawtx_hex: str, mode: str, submitter: Submitter, endpoints: list[str]) -> str:
    if mode == "public":
        return submitter.submit_public(rawtx_hex)
    if mode == "restricted":
        return submitter.submit_restricted(rawtx_hex, endpoints)
    if mode == "hybrid":
        try:
            return submitter.submit_restricted(rawtx_hex, endpoints)
        except Exception:
            return submitter.submit_public(rawtx_hex)
    raise ValueError("Unknown submission mode")

def observe(txid: str, observers: list[Observer]) -> dict:
    mempool_hits = sum(o.seen_in_public_mempool(txid) for o in observers)
    confirmed = next((o.confirmed_height(txid) for o in observers if o.confirmed_height(txid) is not None), None)
    return {"mempool_hits": mempool_hits, "confirmed_height": confirmed}

def zeb_execute(
    intent: dict,
    outcome: EnforcementOutcome,
    psbt_template: dict,
    reveal_s1,
    mode: str,
    submitter: Submitter,
    observers: list[Observer],
    current_tick: int,
    deadline_tick: int,
    get_current_tick,            # callable returning current verified tick
    observer_quorum: int,
    restricted_endpoints: list[str]
) -> ExecutionResult:

    attempt = new_attempt(outcome.intent_hash, outcome.issued_tick)

    if deadline_tick > outcome.expiry_tick:
        return ExecutionResult(
            status="REFUSED",
            execution_id=attempt.attempt_id,
            error_code="E_OUTCOME_MISSING"
        )

    if is_burned(outcome.intent_hash):
        return ExecutionResult(
            status="REFUSED",
            execution_id=attempt.attempt_id,
            error_code="E_BURNED_INTENT"
        )

    if outcome.decision != "ALLOW":
        return ExecutionResult(
            status="REFUSED",
            execution_id=attempt.attempt_id,
            error_code="E_DENIED"
        )

    psbt_final = reveal_s1(psbt_template)
    rawtx_hex = psbt_final["rawtx"]

    txid = submit_transaction(rawtx_hex, mode, submitter, restricted_endpoints)

    while True:
        current_tick = get_current_tick()  # refresh each iteration
        obs = observe(txid, observers)

        if obs["confirmed_height"] is not None:
            return ExecutionResult(
                status="EXECUTED",
                execution_id=attempt.attempt_id,
                rail_result={"txid": txid, "height": obs["confirmed_height"]}
            )

        if exposure_detected(mode, obs["mempool_hits"], observer_quorum):
            burn_attempt(outcome.intent_hash)
            return ExecutionResult(
                status="FAILED",
                execution_id=attempt.attempt_id,
                error_code="E_EXPOSURE_DETECTED"
            )

        if deadline_exceeded(current_tick, deadline_tick):
            burn_attempt(outcome.intent_hash)
            return ExecutionResult(
                status="FAILED",
                execution_id=attempt.attempt_id,
                error_code="E_CONFIRMATION_TIMEOUT"
            )
```

Reference loops are illustrative. Production implementations MUST refresh the current tick from a verified Epoch Clock source on each observation cycle. The `get_current_tick` parameter is a callable that returns the latest verified tick.

`deadline_tick` MUST be less than or equal to `outcome.expiry_tick`. RECOMMENDED: `deadline_tick = outcome.expiry_tick`. If `deadline_tick > outcome.expiry_tick`, the implementation MUST treat the outcome as expired and return `E_OUTCOME_MISSING` (or `E_DENIED` if the consuming system surfaces expiry as a denial reason).

---

## Annex I -- ZET Bridge and Cross-Domain Execution (Normative)

## I.1 Scope

This annex defines cross-chain and Layer-2 execution discipline under ZET.

---

## I.2 BridgeIntent

```python
BridgeIntent = {
  "intent_id": "tstr",
  "source_domain": "tstr",
  "destination_domain": "tstr",
  "asset_ref": "tstr",
  "issued_tick": "uint",
  "expiry_tick": "uint",
  "suite_profile": "tstr",
  "signature": "bstr",
  "session_id": "bstr(16)",
  "exporter_hash": "bstr",
  "decision_id": "tstr"
}
```

### I.2A BridgeIntent Signature Preimage (Normative)

BridgeIntent MUST be deterministic CBOR.

`signature` MUST be computed over the deterministic CBOR encoding of BridgeIntent with the `signature` field omitted.

Verification MUST reconstruct identical canonical bytes and verify the signature under the declared `suite_profile`. Unknown `suite_profile` values MUST cause rejection.

---

## I.3 Execution Rules

1. No executable artefact exists before approval.
2. Execution MUST be multi-phase:

   * lock
   * attest
   * release
3. All phases MUST be bound to the same session_id, exporter_hash, and decision_id.
4. Optimistic trust assumptions MUST NOT be used.

---

## Implementation Checklist (Informative)

* [ ] ZET boundary with replay protection
* [ ] Epoch Clock integration (not system clock)
* [ ] Burn discipline with persistent storage
* [ ] Exposure detection with configurable quorum
* [ ] S1 revelation only after enforcement approval
* [ ] No executable artefacts before PQSEC approval
* [ ] Cross-domain binding (if implementing bridges)
* [ ] Thread-safe attempt identity generation

---

## Annex J -- ObservationReport (Informative)

### J.1 Purpose

Observation reports summarise what was observed during transaction broadcast, providing evidence for post-broadcast analysis.

### J.2 Receipt Type

**ReceiptEnvelope.type:** `"zeb.observation_report"`

**Body:**

| Field | Type | Description |
|-------|------|-------------|
| `v` | uint | Schema version |
| `txid` | bstr (32) | Transaction ID observed |
| `first_seen_tick` | uint | When first observed |
| `observer_count` | uint | Number of observers |
| `propagation_estimate` | tstr | Estimated propagation |
| `anomalies` | array of tstr | Detected anomalies |

`v` is the ObservationReport schema version. For this specification, `v` MUST be 1.

### J.3 Authority Boundary

Observation reports are informative only. They do NOT grant authority.

---

## Execution Profile Evidence (Informative)

For profiles requiring observation:

Implementations SHOULD emit an ObservationReport and make it available as evidence for PQSEC Annex AC evaluation when required by policy.

**Recommended receipt types:**
- `zeb.observation_report`

ZEB receipts are audit artefacts only and MUST NOT be treated as permission signals.

---

## Receipt Integration

Implementations MAY emit ReceiptEnvelope objects as defined in PQSF Annex W.

Where a receipt asserts an enforcement decision, the receipt type MUST be one of the PQSEC Audit Receipts defined in PQSEC Annex AE.

Domain receipts (observation, gate, submission, fulfilment) MUST NOT be used to replace required protocol logic and MUST be treated as audit artefacts only.

Receipt presence MUST NOT imply permission. Only PQSEC evaluation determines authority.

---

## Changelog

### Version 1.3.0

**Execution Boundary Clarified**

* Formalised ZET packaging statement.
* Explicitly defined ZET as Part I of ZEB with no independent authority surface.
* Strengthened authority boundary language across both parts.

**Scope Hardening**

* Added Canonical Scope and Limitation Disclaimer (1A).
* Explicitly disclaimed censorship resistance, replacement-proof claims, and quantum immunity.

**Replay Discipline Strengthened**

* EnforcementOutcome structure standardised (decision vs allowed clarified).
* Replay guard made mandatory with explicit error signalling.

**Execution Discipline Formalised**

* Prohibited pre-approval material injection.

**Burn Semantics Clarified**

* Mandatory burn discipline for exposure, timeout, and invalidation.
* Persistent burned intent tracking required.

**Bridge Execution Rules Formalised**

* Lock--attest--release multi-phase rule added.
* Explicit cross-domain binding to session_id, exporter_hash, and decision_id.

**Error Codes Normalised**

* Consolidated deterministic error signalling.
* Explicit normative error code list added.

**Dependency Versions Updated**

* Epoch Clock minimum version increased.
* Explicit PQSF dependency clarified for session binding.

---

If you find this work useful and wish to support continued development, donations are welcome:

**Bitcoin:**
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
