# Direct verification

The exact Direct Mode command is:

```text
PYTHONIOENCODING=utf-8 python -m pytest tests -q
```

Result: `12 passed`.

Covered cases:

- empty and registered state plus JSON readback;
- owner authorization and date-role ordering;
- Gregorian invalid-date rejection, year-zero rejection, and valid leap-day acceptance;
- source URI/hash registration, digest mismatch rejection, and stale source-window rejection before consensus;
- `SEAL_OK` consensus, validator agreement, and validator disagreement;
- pre-implementation, implementation-window, effective-day, and grandfathering boundaries;
- cohort mismatch to `AUTHORITY_UNCLEAR`;
- escaped delimiter-breakout/prompt-injection and malformed notice rejection without sealing;
- stale revision replay and explicit supersession;
- same-day/same-cohort rollout replay rejection.

The exact linter command is:

```text
PYTHONIOENCODING=utf-8 genvm-lint check contracts/public_program_transition_day_gate.py
```

Result: lint and SDK validation passed. Studionet deployment and the complete E2E evidence are recorded in [`e2e-matrix.md`](e2e-matrix.md); this file describes the local Direct Mode coverage.

The user-provided `RESEARCH.md` remains outside the candidate package and is explicitly excluded by the project `.gitignore`; it is not a deployable contract artifact.
