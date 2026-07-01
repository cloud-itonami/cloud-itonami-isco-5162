# cloud-itonami-isco-5162

Open Occupation Blueprint for **ISCO-08 5162**: Companions and Valets.

This repository designs a forkable OSS business for an independent companion/valet: a mobility-support robot performs supply setup and light-errand tasks under a governor-gated actor, so the practice keeps its own service and consent records instead of renting a closed companion-care SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a mobility-support robot performs supply setup and light-errand carrying tasks alongside a supervising companion under an actor that proposes
actions and an independent **Companion Valet Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
lifting/transfer assist, or medication reminders) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
client consent + care plan + service scope
        |
        v
Companion Advisor -> Companion Valet Governor -> assist/log, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `5162`). Required capabilities:

- :robotics
- :identity
- :forms
- :audit-ledger
- :bpmn

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
