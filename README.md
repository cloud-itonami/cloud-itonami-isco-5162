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

## Reference implementation (`:maturity :implemented`)

Full itonami Actor pattern (per ADR-2607011000 / CLAUDE.md's Actors
section, alongside `cloud-itonami-isco-6130`, `-8160`, `-2166`, `-2641`,
`-2651`, `-2652`, `-2654`, `-1219`, `-1223`, `-1330`, `-1341`, `-1349`,
`-1412`, `-1439`, `-2144`, `-2320`, `-2411`, `-2422`, `-2431`, `-2621`,
`-2634`, `-3122`, `-3123`, `-3141`, `-3255`, `-3339`, `-3512`, `-4120`,
`-4131`, `-4132`, `-4211`, `-4224`, `-4229`, `-4322`, `-4413`, `-4415`
and `-5120`): a real
[`kotoba-lang/langgraph`](https://github.com/kotoba-lang/langgraph)
`StateGraph`, with the Advisor and Governor as distinct graph nodes and
human-in-the-loop interrupt/resume via checkpointing.

```text
:intake -> :advise -> :govern -> :decide -+-> :commit            (:ok? true)
                                           +-> :request-approval   (:escalate? true, interrupt-before)
                                           +-> :hold               (:hard? true)
```

- `src/companion_valet/store.kotoba` — `Store` protocol +
  `MemStore`: registered clients, committed records, an append-only
  audit ledger.
- `src/companion_valet/advisor.kotoba` — `Advisor` protocol;
  `mock-advisor` (deterministic, default) proposes an assist or log
  operation from a request; `llm-advisor` wraps a
  `langchain.model/ChatModel` — either way the advisor only ever
  produces a `:propose`-effect proposal, never a committed record, and
  LLM parse failures always yield `confidence 0.0` (forces escalation,
  never fabricated confidence).
- `src/companion_valet/governor.kotoba` —
  `CompanionValetGovernor/check`: a pure function, wired as its own
  `:govern` node. Hard invariants (unregistered client, a proposal
  whose `:effect` isn't `:propose`) always route to `:hold`. Escalation
  invariants (`:lifting-transfer-assist`, `:medication-reminder`, or
  low advisor confidence) always route to `:request-approval` — an
  `interrupt-before` node that the graph checkpoints and only resumes
  on explicit human approval (`actor/approve!`), matching the README's
  robotics-premise statement that lifting/transfer assist or
  medication reminders always require human sign-off.
- `src/companion_valet/actor.kotoba` — `build-graph`, `run-request!`,
  `approve!`: the `langgraph.graph/state-graph` wiring itself.

```bash
clojure -M:test
```

This is what backs this repo's `:maturity :implemented` entry in
[`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation).

## License

AGPL-3.0-or-later.
