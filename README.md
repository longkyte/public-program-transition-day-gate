# Public Program Transition Day Gate

Consensus-backed registration and evaluation of a public-program rule transition when publication, implementation, effective, and grandfathering dates have different roles.

## Live Deployment

Network: Studionet, Chain ID `61999`.

Primary release instance: `0x6058B19351f0bB3C7fbbdDe129cbbD1eA5bf8671`.
Deployer: `0x5B6465eD6Ec0F2F7944b8279E8872123bf9b545a`.
[Contract Explorer](https://explorer-studio.genlayer.com/address/0x6058B19351f0bB3C7fbbdDe129cbbD1eA5bf8671) · [deploy transaction](https://explorer-studio.genlayer.com/tx/0x35e73902ed54284de77331169210b94303fd2d437a8a9199343922c1821bec08).

The deploy transaction is `FINALIZED` with GenVM `SUCCESS`. The complete 17-row integration/consensus matrix, including authoritative readbacks and isolated negative cases, is in [`verification/e2e-matrix.md`](verification/e2e-matrix.md). Representative evidence: [successful notice seal](https://explorer-studio.genlayer.com/tx/0xf5b9812c6e5f271f22823338e5f773936292286b8cd5f7b6bdd5d739a16471b9) (`SEAL_OK`, 4/5 agree, `FINALIZED/SUCCESS`), [cohort fail-closed evaluation](https://explorer-studio.genlayer.com/tx/0x81635601457f63a85b10273d1b382690659339b2c7582f222e28b475c71d80a3) (`AUTHORITY_UNCLEAR`, `FINALIZED/SUCCESS`), and [prompt-injection rejection](https://explorer-studio.genlayer.com/tx/0xedcc5d4ab51845ca9b372362f9402894403867d959240762162cd32a9214ff40) (`NOTICE_UNVERIFIED`, `SEAL_REJECT`, `FINALIZED` rollback).

The final primary readback after E11/E15 is `AUTHORITY_UNCLEAR`, reason `COHORT_MISMATCH`, lifecycle `PENDING`, with transition and sealed notice revision `3`. The isolated negative-test instance is `0xE5814389877119e0db25572648de437450A7C4b0`; the separate E16 empty-state evidence instance is `0x9f71EaCEA3C206c50b37061D629C5838AD278E18`. Both are linked from the matrix and neither is the release instance.

## Problem and Why GenLayer

Public programs often publish a new rule before operators implement it, make it effective, or end a grandfathering window. Treating those dates as one date can activate the wrong rule for a cohort. This contract stores the four roles separately, asks validators to reach consensus on untrusted official-notice evidence, then derives a deterministic rule pointer for one rollout day and cohort. It never decides eligibility.

A conventional backend is sufficient when one trusted operator owns the source and a single deterministic scheduler can be accepted. GenLayer is useful here when independent validators must inspect the notice meaning and reject ambiguity, stale amendments, malformed output, or prompt-injection text before a consequential transition pointer is stored.

## How It Works

1. The owner registers authority, program, cohort, old/new versions, four date roles, exception policy, notice revision, supersession revision, source URI/hash, source publication/valid-until days, and one optional amendment notice with its source URI.
2. The owner seals the notice for an explicit `as_of_day`. The contract rejects an as-of day outside the registered source freshness window before any LLM call. A leader and validators independently inspect the escaped notice evidence with a bounded JSON-only decision. Only `SEAL_OK` can move the lifecycle to `NOTICE_SEALED`.
3. Any rollout workflow can evaluate a day and cohort. The contract derives `NEW_RULE_ACTIVE`, `OLD_RULE_TRANSITION`, `NOT_YET_ACTIVE`, or fail-closed `AUTHORITY_UNCLEAR` without deciding eligibility.

## State Model and Invariants

- Lifecycle starts at `EMPTY` and moves through `REGISTERED` and `NOTICE_SEALED`.
- Evaluation derives `ACTIVATED`, `TRANSITION`, or `PENDING`; a superseding registration returns the record to `REGISTERED`.
- `publication_day <= implementation_day <= effective_day` is required.
- `GRANDFATHERED` requires a grandfathering boundary on or after the effective day.
- Date fields use strict Gregorian calendar validation, including leap years and rejection of year `0000`.
- Old and new rule versions must differ.
- Notice revisions strictly increase. A new registration must explicitly name the current transition revision it supersedes.
- A primary source URI and 64-hex SHA-256 hash of the canonical UTF-8 primary notice are required; the contract recomputes and compares the digest before consensus. Source publication and valid-until days form a freshness window, and `seal_notice(as_of_day)` must be inside it.
- A same-day, same-cohort evaluation cannot be replayed.
- A cohort mismatch produces `AUTHORITY_UNCLEAR` and never activates a rule.

## Public API

- `register_transition(...)`: owner-only registration or explicit supersession.
- `seal_notice(as_of_day)`: owner-only freshness check, nondeterministic notice review, and consensus seal.
- `evaluate_rollout(rollout_day, requested_cohort_id)`: deterministic state transition for a rollout workflow; no eligibility decision.
- `read_status()`: lifecycle view.
- `read_active_pointer()`: oracle view for downstream registries, schedulers, and compliance workflows.
- `read_transition()`: deterministic JSON view of the bound record and last evaluation.

## Consensus Binding Matrix

| Field | Source | Stored? | Downstream effect | Validator check | Binding mode | Differential test |
| --- | --- | --- | --- | --- | --- | --- |
| `SEAL_OK` / `SEAL_REJECT` | Leader/validator inspection of notices | `NOTICE_SEALED` only after `SEAL_OK` | Enables evaluation | Both independently return the same bounded decision; malformed output rejects | Exact enum agreement | `SEAL_OK` vs `SEAL_REJECT` validator disagreement |
| authority/program/cohort | Registration plus notice evidence | Yes | Scope of the transition | Notice must explicitly support each value | Exact field match in the consensus prompt | Cohort mismatch yields `AUTHORITY_UNCLEAR` |
| old/new rule version | Registration plus notice evidence | Yes | Pointer names the version boundary | Notice must support both and they must differ | Exact field match and deterministic storage | Old == new rejected |
| publication/implementation/effective/grandfathering day | Registration plus notice evidence | Yes | Date-role state machine | Notice must support every role; Gregorian validity and deterministic ordering checks | Exact string fields; lexical ISO-day comparisons | Invalid 2027-02-29, valid 2028-02-29, and boundary tests |
| exception policy | Registration plus notice evidence | Yes | Grandfathering behavior | Notice must support the policy; grandfathering window is checked deterministically | Exact enum plus deterministic derivation | Grandfathered cohort remains in transition until boundary |
| notice revision/supersession | Registration | Yes | Replay and amendment control | Strict monotonic revision and explicit predecessor | Deterministic exact integers | Stale replay and wrong predecessor revert |
| source URI/hash and freshness window | Registered notice metadata plus notice evidence | Yes | Blocks stale or altered source sealing | Exact URI/hash/revision/window must be supported; digest is recomputed from canonical UTF-8 notice bytes and `as_of_day` must be inside the window | Deterministic digest comparison and pre-consensus freshness gate | Stale as-of and mutated-notice/same-hash reverts |
| notice text | Caller-supplied evidence | Yes | Consensus seal only | Entity-escaped boundary prevents tag breakout; embedded instructions are ignored | Escaped untrusted-data fields | Closing-tag breakout regression |
| active pointer | Derived from sealed fields and requested cohort/day | Yes | Downstream oracle result | Not LLM-authored; deterministic from bound fields | Deterministic derivation | Before, transition, effective, mismatch matrix |

## Security and Failure Behavior

Notice content is enclosed in explicit escaped untrusted-data tags. Ampersands and angle brackets are entity-escaped before interpolation, so a caller-supplied closing tag cannot terminate the evidence boundary. Embedded instructions, role claims, and format requests are ignored by the review prompt. The parser accepts only the exact one-key JSON decision and maps malformed output to rejection. No notice is sealed on `SEAL_REJECT` or validator disagreement. Owner checks protect registration and sealing. Date, source metadata, identifier, version, notice-size, revision, supersession, replay, and lifecycle checks fail closed with stable error codes.

The contract stores only the bounded decision and the registered evidence fields; it does not store an LLM explanation or allow explanation wording to affect state. It is intentionally not a payout, eligibility, or enforcement contract.

## Tests

Install the exact pinned dependencies and run:

```text
python -m pytest tests -q
genvm-lint check contracts/public_program_transition_day_gate.py
```

The Direct Mode suite covers initial storage, owner authorization, Gregorian date ordering and leap-year validity, source freshness, consensus agreement and disagreement, lifecycle boundaries, grandfathering, cohort mismatch, delimiter-breakout/prompt-injection rejection, malformed consensus output, stale revision replay, explicit supersession, and rollout replay prevention. Direct Mode uses strict nondeterministic-call mocks; the completed Studionet integration/consensus evidence is recorded in [`verification/e2e-matrix.md`](verification/e2e-matrix.md).

## Consensus Engineering Lessons

- Store a bounded consensus decision, not free-form LLM prose.
- Keep notice evidence untrusted, escaped, and delimited even when the caller is an official operator.
- Bind notice source identity and a bounded freshness window into storage and the consensus prompt; reject stale as-of days deterministically.
- Separate publication, implementation, effective, and grandfathering dates so each has one deterministic consequence.
- Require explicit revision supersession instead of silently accepting stale amendments.
- Treat a cohort mismatch as `AUTHORITY_UNCLEAR`, never as an implicit default cohort.
- Evaluate state only after a finalized consensus seal; this implementation still requires on-chain finality and authoritative readback before release claims.

## Reusable Integrations

1. A rules registry can call `read_active_pointer()` before applying a versioned policy.
2. A rollout scheduler can call `evaluate_rollout()` for a bounded day/cohort and persist the resulting pointer in its own workflow.
3. A compliance or appeals workflow can call `read_transition()` to audit the exact authority, notice revision, date roles, exception policy, and last evaluation.

## Limitations

- The contract treats the supplied notice strings and source metadata as evidence; an operator or integration must source and reproduce official notices correctly. The contract verifies the notice digest, binds the claimed source identity and freshness window, but cannot fetch the source itself.
- Notice interpretation depends on validator LLM behavior and can remain unsealed when validators disagree or the evidence is ambiguous.
- Date strings use strict ISO `YYYY-MM-DD` validation, year `>= 0001`, and Gregorian month/day bounds; timezone conversion belongs outside this primitive.
- No eligibility, payout, appeal, or external contract message is performed.
- Studionet integration evidence is not claimed until the exact approved revision is deployed and the full E2E matrix passes.

## Repository Structure

```text
contracts/public_program_transition_day_gate.py
tests/test_public_program_transition_day_gate.py
samples/transition-register.json
samples/notice-sealed.json
samples/negative-cases.json
verification/test-summary.json
verification/test-cases.md
verification/e2e-matrix.md
README.md
requirements.txt
LICENSE
.gitignore
```

## License

MIT
