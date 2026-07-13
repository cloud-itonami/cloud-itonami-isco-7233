# cloud-itonami-isco-7233

Open Occupation Blueprint for **ISCO-08 7233**: Agricultural and Industrial Machinery Mechanics and Repairers.

**Maturity: `:implemented`** — RepairAdvisor ⊣ MachineryRepairGovernor
as a langgraph StateGraph (`intake → advise → govern → decide →
commit/hold`, human-approval interrupt), modeled on
cloud-itonami-isco-4311's bookkeeping actor. 14 tests / 29 assertions
green. The governor never dispatches hardware — it only gates what
the diagnostic/lift-assist robot below may execute.

The repair HARD invariants — set membership and arithmetic, not
discretion:

1. **Approved-part membership** — the proposed replacement part must
   be a member of the equipment's registered approved-parts set (no
   counterfeit or unauthorized part).
2. **Test-deviation ceiling** — the proposed post-repair test
   deviation must not exceed the equipment's registered ceiling.

`:approve-lift-operation` and `:approve-hydraulic-pressure-work`
**always** escalate to human sign-off regardless of confidence, per
this repo's Trust Controls (business-model.md).

This repository designs a forkable OSS business for an independent farm and industrial machinery mechanic: a diagnostic and lift-assist robot performs equipment scanning and part-handling under a governor-gated actor, so the practice keeps its own service records instead of renting a closed field-service SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a diagnostic and lift-assist robot performs equipment scanning and part-handling for machinery repair under an actor that proposes
actions and an independent **Machinery Repair Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
operating a lift, or working with hydraulic systems under pressure) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
service request + equipment history + repair scope
        |
        v
Repair Advisor -> Machinery Repair Governor -> repair/test, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7233`). Required capabilities:

- :robotics
- :forms
- :telemetry
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
