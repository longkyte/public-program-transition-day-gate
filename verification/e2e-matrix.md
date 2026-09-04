# Studionet E2E scenario matrix

This matrix records the completed Studionet E2E run. Primary final instance: `0x6058B19351f0bB3C7fbbdDe129cbbD1eA5bf8671`; isolated negative-test instance: `0xE5814389877119e0db25572648de437450A7C4b0`; approved source revision: `PP-TDG-PREDEPLOY-f7991a7084f7652eb60147cd80fa4096cdbdde41673e8c649044cb8b22068a10`.

`E2E COMPLETION GATE: PASS`

| ID | Scenario and exact input | Pre-state / isolation | Expected result and state | Evidence required | Status |
| --- | --- | --- | --- | --- | --- |
| E01 | Deploy with no constructor args | Fresh primary deployment `0x6058B19351f0bB3C7fbbdDe129cbbD1eA5bf8671` | Constructor `SUCCESS`; initial lifecycle `EMPTY`; pointer `PENDING` | [deploy tx](https://explorer-studio.genlayer.com/tx/0x35e73902ed54284de77331169210b94303fd2d437a8a9199343922c1821bec08); `FINALIZED`; source parity; initial views | PASS |
| E02 | Register `samples/transition-register.json` | E01 state | `REGISTERED`, revision `1`, pointer `PENDING` | [tx](https://explorer-studio.genlayer.com/tx/0xb18204813075550ac798ef1761268217d06a0f17a08c0e524574118eb1b9c8e0); `FINALIZED/SUCCESS`; readback revision 1 | PASS |
| E03 | `seal_notice("2027-01-15")` with the primary notice in `samples/notice-sealed.json` | E02 state; as-of day inside source window | Consensus success; `NOTICE_SEALED`; sealed revision `1` | [tx](https://explorer-studio.genlayer.com/tx/0xf5b9812c6e5f271f22823338e5f773936292286b8cd5f7b6bdd5d739a16471b9); `FINALIZED/SUCCESS`; `SEAL_OK`; 4/5 agree; readback | PASS |
| E04 | `evaluate_rollout("2026-12-31", "cohort-2027")` | E03 state | `NOT_YET_ACTIVE`; reason `BEFORE_PUBLICATION` | [tx](https://explorer-studio.genlayer.com/tx/0xe0bd3025804a8ab1e27fb5f157ea37cc2b4ce7b1c585c4a75363ea21dfdea293); `FINALIZED/SUCCESS`; readback pointer/reason | PASS |
| E05 | `evaluate_rollout("2027-02-01", "cohort-2027")` | E04 state; new day avoids replay | `OLD_RULE_TRANSITION`; lifecycle `TRANSITION` | [tx](https://explorer-studio.genlayer.com/tx/0xa50b0c2c13e8db2bc3e2a0e560f15dd95b8cabf8007045e8d09b5e5a62a87257); `FINALIZED/SUCCESS`; readback | PASS |
| E06 | `evaluate_rollout("2027-03-01", "cohort-2027")` | E05 state; new day | `NEW_RULE_ACTIVE`; lifecycle `ACTIVATED` | [tx](https://explorer-studio.genlayer.com/tx/0xf753a666bc8072d4518c55d25a420457d9ba1d70003d4e79d33436952cb2c688); `FINALIZED/SUCCESS`; 5/5 agree; readback | PASS |
| E07 | Register revision `2` with amendment/source and `supersedes_revision=1`, then seal `2027-01-15` | E06 state | `REGISTERED` then `NOTICE_SEALED`; revision `2` | [register](https://explorer-studio.genlayer.com/tx/0x905c66cdf16b14cd0c62e86b32b928aef232d18f17dfe61788eed712166d840e) and [seal](https://explorer-studio.genlayer.com/tx/0x96ba87be831f70faff4e7aa6001182ba969c2283ea290358a39c4e521fbfdf42); both `FINALIZED/SUCCESS`; `SEAL_OK`; readback | PASS |
| E08 | Register `GRANDFATHERED` revision `3` with valid window, then seal | E07 state; explicit supersession | Registration and seal succeed; revision `3` | [register](https://explorer-studio.genlayer.com/tx/0x659354755a3eef7528e204c2d9162abba44406a867f709eabafd56c6ebcfca31) and [seal](https://explorer-studio.genlayer.com/tx/0xe81e4c813dd8d51af84e9c5314462bc34585dd95d8dce5449adc2cd462acc026); `FINALIZED/SUCCESS`; readback sealed revision 3 | PASS |
| E09 | `evaluate_rollout("2027-03-15", "cohort-2027")` | E08 state | `OLD_RULE_TRANSITION`; reason `GRANDFATHERING_WINDOW` | [tx](https://explorer-studio.genlayer.com/tx/0x322ff507615d037c70b890819ca6ce442f5aeb272aa6544642901010bc0b46bf); `FINALIZED/SUCCESS`; 5/5 agree; readback | PASS |
| E10 | `evaluate_rollout("2027-04-01", "cohort-2027")` | E09 state; new day | `NEW_RULE_ACTIVE` | [tx](https://explorer-studio.genlayer.com/tx/0x56a061f09a45d9f76f3e5c72982ebcbeb5ed0361973173c286dab039fc2f47ad); `FINALIZED/SUCCESS`; readback | PASS |
| E11 | `evaluate_rollout("2027-04-02", "other-cohort")` | E10 state; new day | `AUTHORITY_UNCLEAR`; lifecycle `PENDING` | [tx](https://explorer-studio.genlayer.com/tx/0x81635601457f63a85b10273d1b382690659339b2c7582f222e28b475c71d80a3); `FINALIZED/SUCCESS`; readback proves no activation | PASS |
| E12 | Re-submit revision `3` | E11 state | Deterministic rollback `STALE_NOTICE_REVISION`; state unchanged | [tx](https://explorer-studio.genlayer.com/tx/0x196bd125a0da456a5abafd9f7784cb4deb0cdac11ba082b1c09f8b7fe52d4cbe); `FINALIZED`; GenVM `ERROR/Rollback`; error and after-readback | PASS |
| E13 | `seal_notice("2027-02-16")` with valid-until `2027-02-15` | Isolated registered instance `0xE5814389877119e0db25572648de437450A7C4b0` | Deterministic rollback `STALE_NOTICE_SOURCE`; state unchanged | [tx](https://explorer-studio.genlayer.com/tx/0xc66f0b6187880d70ab55242d47f26c2caf084dc28ab89d17f10fe532ca646b79); `FINALIZED`; error and authoritative readback | PASS |
| E14 | Seal escaped delimiter-breakout/injected notice from `samples/negative-cases.json` | Isolated registered revision 2 | Consensus rejection; no `NOTICE_SEALED`; state unchanged | [seal tx](https://explorer-studio.genlayer.com/tx/0xedcc5d4ab51845ca9b372362f9402894403867d959240762162cd32a9214ff40); `FINALIZED`; `NOTICE_UNVERIFIED`; `SEAL_REJECT`; readback | PASS |
| E15 | Re-evaluate exact `2027-04-02 / other-cohort` | E11 state | Deterministic rollback `REPLAYED_ROLLOUT`; state unchanged | [tx](https://explorer-studio.genlayer.com/tx/0x44226ef6a865eafc717c697cdfd3429e7417859eaed7425837b6735d20353a38); `FINALIZED`; error and readback | PASS |
| E16 | Register from non-owner `0xF181A48BC9058B1F02A48d2ca38f292933E5C051` | Fresh evidence-isolated deployment `0x9f71EaCEA3C206c50b37061D629C5838AD278E18` ([deploy](https://explorer-studio.genlayer.com/tx/0x8ac99d88b49418aecdd8f5e1cac3141d24b9456c3800c791029641f334cd4553)) | Deterministic rollback `UNAUTHORIZED`; no state change | [unauthorized tx](https://explorer-studio.genlayer.com/tx/0x2b5cf757ad7a45483f920241a940187fccebbbd1f06e2f7f72b85ece40926e74); `FINALIZED`; error and empty-state evidence | PASS |
| E17 | Register mutated primary notice while retaining old `notice_hash` | Isolated injected revision state | Deterministic rollback `INVALID_SOURCE_HASH`; state unchanged before consensus | [tx](https://explorer-studio.genlayer.com/tx/0x9848a49220936d4ae809b9bec53fdf44cbb125a389e69207a54dee4609365e30); `FINALIZED`; error and authoritative readback | PASS |

## Authoritative readback register

The following are the exact `read_transition()` snapshots captured after each consequential primary/isolated operation. For rollback rows, the listed state is the post-transaction snapshot and is the unchanged state required by the scenario.

| ID | Instance | Exact post-state / unchanged-state readback |
| --- | --- | --- |
| E01 | Primary `0x6058B19351f0bB3C7fbbdDe129cbbD1eA5bf8671` | `lifecycle=EMPTY`, `active_pointer=PENDING`, `reason_code=UNREGISTERED`, `transition_revision=0`, `sealed_notice_revision=0` |
| E02 | Primary | `lifecycle=REGISTERED`, `active_pointer=PENDING`, `reason_code=REGISTERED`, `notice_revision=1`, `transition_revision=1`, `sealed_notice_revision=0`, `notice_hash=17f7dedc71b5fb3741d2f2b096b0e1de7f602a7e4b0883c23b2b29bee6ae7b72` |
| E03 | Primary | `lifecycle=NOTICE_SEALED`, `active_pointer=PENDING`, `reason_code=NOTICE_SEALED`, `notice_revision=1`, `sealed_notice_revision=1` |
| E04 | Primary | `lifecycle=PENDING`, `active_pointer=NOT_YET_ACTIVE`, `reason_code=BEFORE_PUBLICATION`, `last_rollout_day=2026-12-31`, `last_evaluated_cohort=cohort-2027` |
| E05 | Primary | `lifecycle=TRANSITION`, `active_pointer=OLD_RULE_TRANSITION`, `reason_code=IMPLEMENTATION_WINDOW`, `last_rollout_day=2027-02-01`, `last_evaluated_cohort=cohort-2027` |
| E06 | Primary | `lifecycle=ACTIVATED`, `active_pointer=NEW_RULE_ACTIVE`, `reason_code=EFFECTIVE_DATE_REACHED`, `last_rollout_day=2027-03-01`, `last_evaluated_cohort=cohort-2027` |
| E07 | Primary | `lifecycle=NOTICE_SEALED`, `active_pointer=PENDING`, `notice_revision=2`, `transition_revision=2`, `sealed_notice_revision=2`, `exception_code=AMENDMENT`, amendment source retained |
| E08 | Primary | `lifecycle=NOTICE_SEALED`, `active_pointer=PENDING`, `notice_revision=3`, `transition_revision=3`, `sealed_notice_revision=3`, `exception_code=GRANDFATHERED` |
| E09 | Primary | `lifecycle=TRANSITION`, `active_pointer=OLD_RULE_TRANSITION`, `reason_code=GRANDFATHERING_WINDOW`, `last_rollout_day=2027-03-15`, `last_evaluated_cohort=cohort-2027` |
| E10 | Primary | `lifecycle=ACTIVATED`, `active_pointer=NEW_RULE_ACTIVE`, `reason_code=EFFECTIVE_DATE_REACHED`, `last_rollout_day=2027-04-01`, `last_evaluated_cohort=cohort-2027` |
| E11 | Primary | `lifecycle=PENDING`, `active_pointer=AUTHORITY_UNCLEAR`, `reason_code=COHORT_MISMATCH`, `last_rollout_day=2027-04-02`, `last_evaluated_cohort=other-cohort` |
| E12 | Primary | Unchanged from E11: `lifecycle=PENDING`, `active_pointer=AUTHORITY_UNCLEAR`, `reason_code=COHORT_MISMATCH`, `notice_revision=3`, `sealed_notice_revision=3`, last day/cohort remain `2027-04-02 / other-cohort` |
| E13 | Isolated `0xE5814389877119e0db25572648de437450A7C4b0` | Unchanged registered state: `lifecycle=REGISTERED`, `active_pointer=PENDING`, `reason_code=REGISTERED`, `notice_revision=1`, `transition_revision=1`, `sealed_notice_revision=0` |
| E14 | Isolated `0xE5814389877119e0db25572648de437450A7C4b0` | Unchanged after consensus rejection: `lifecycle=REGISTERED`, `active_pointer=PENDING`, `notice_revision=2`, `transition_revision=2`, `sealed_notice_revision=0`; injected notice remains unsealed |
| E15 | Primary | Unchanged from E11: `lifecycle=PENDING`, `active_pointer=AUTHORITY_UNCLEAR`, `reason_code=COHORT_MISMATCH`, last day/cohort remain `2027-04-02 / other-cohort` |
| E16 | Evidence-isolated `0x9f71EaCEA3C206c50b37061D629C5838AD278E18` | Unchanged empty state after non-owner rollback: `lifecycle=EMPTY`, `active_pointer=PENDING`, `reason_code=UNREGISTERED`, `notice_revision=0`, `transition_revision=0`, `sealed_notice_revision=0`; deployment tx [here](https://explorer-studio.genlayer.com/tx/0x8ac99d88b49418aecdd8f5e1cac3141d24b9456c3800c791029641f334cd4553) |
| E17 | Isolated `0xE5814389877119e0db25572648de437450A7C4b0` | Unchanged from E14: `lifecycle=REGISTERED`, `active_pointer=PENDING`, `notice_revision=2`, `transition_revision=2`, `sealed_notice_revision=0`; old digest retained and no consensus call made |

## Safe execution and failure harvesting

The primary E01 deployment is the release instance and E02-E12/E15 use it. E07-E10 reuse the same instance through explicit revisions. The first isolated deployment supports E13, E14, and E17; a second evidence-isolated deployment supports E16 so its empty post-rollback readback is independently reconstructible. Neither isolated deployment replaces E01 evidence. The first isolated deployment tx is `0xf01687a2b6dfc2d92e10a7ef0d92d0595b7e558601d332cf70636ed3a65389af`; the E16 evidence-isolated deployment tx is `0x8ac99d88b49418aecdd8f5e1cac3141d24b9456c3800c791029641f334cd4553`.

If a scenario fails, freeze the deployed revision and record failure ID, scenario ID, exact input, pre-state, tx hash, receipt/result, expected-vs-actual, readback, and Explorer URL. Continue only independent rows. Mark dependent rows `BLOCKED BY <FAILURE_ID>` without sending unsafe writes. Group findings by root cause, apply one batch correction, rerun lint/direct/validator checks, obtain a new PRE-DEPLOY approval, deploy once, and rerun the complete matrix.

## RPC efficiency record

- Deployments: `3` total for one approved revision: one primary release instance, one isolated negative-test instance for E13/E14/E17, and one evidence-isolated instance for E16's authoritative empty-state readback.
- Logical writes: one transaction per executed row; E06 was retried only after the first attempts were confirmed not broadcast by Explorer and returned RPC cooldown errors. No duplicate logical write was accepted.
- Polling: bounded CLI/Explorer checks with backoff until terminal `FINALIZED`; no concurrent pollers.
- Readback: one authoritative `read_transition` snapshot after each state-changing success/revert; cached receipt/hash/Explorer URL reused in this matrix.
- Isolation: primary state reused for E01-E12/E15; isolated instance used where reuse would hide authorization, stale-source, malformed-consensus, or digest failures.
- Failure harvesting: the initial terse-notice failure was recorded on a diagnosis instance, excluded from release evidence, corrected in the fixture, re-reviewed, redeployed once, and the complete final matrix was rerun.
- E2E completion: all E01-E17 are `PASS`; no row remains `NOT RUN`, `PENDING`, `PARTIAL`, or `ASSUMED`.
