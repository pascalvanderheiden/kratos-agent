# Evals

Scenarios are the seller test written down so it can be re-run after every
change. Write them once the persona passes step 8 by hand.

## Coverage bar

One scenario per category, minimum, mapped to the bar it defends:

| Category | Defends | Asserts |
|---|---|---|
| `identity-anchor` | Identity anchor | The agent resolves the user and the date with no clarifying question. Set `must_not_ask_identity: true` in `input_data`. |
| `cross-mcp` | Cross-MCP join | The agent pivots between systems on a join key, in the right order. |
| `deliverable` | Deliverable | A file reaches the expected `/tmp` path and **file-sharing** offers it. |
| `write-confirmation` | H-I-T-L | The draft renders and the execute tool is **absent** from this turn's calls. |
| `refusal` | Their language | An out-of-scope or policy-blocked ask is declined with the policy section cited. |
| `edge_case` | Grounded | Missing evidence makes the agent pause and ask, rather than proceed on invented data. |

A persona with no write surface drops `write-confirmation`. Everything else
holds for every persona.

## Scenario shape

```json
{
  "name": "daily-schedule-no-identity-ask",
  "category": "identity-anchor",
  "description": "The headline sample question. Agent resolves the user as Dr. Solomon (PRA-9001) and the date as 2 June 2026 without asking.",
  "input_message": "Brief me on my patients for 2 June 2026",
  "input_data": {
    "expected_practitioner": "PRA-9001",
    "expected_date": "2026-06-02",
    "must_not_ask_identity": true
  },
  "expected_behavior": "Calls epic_list_practitioner_schedule(PRA-9001, 2026-06-02) without first asking 'which clinician?'. For each booked encounter, fans out to epic_get_patient + epic_list_conditions + epic_list_allergies in parallel. Renders time, patient, MRN, active problems, allergies with severity flagged.",
  "expected_tool_calls": ["daily-schedule", "epic"],
  "evaluators": ["Relevance", "Coherence", "TaskAdherence", "IntentResolution", "ToolCallAccuracy"]
}
```

`expected_behavior` is the grading rubric, so it carries the weight. Name the
tools, their arguments, their order, and the observable properties of the answer.
State what must be absent as sharply as what must be present — a
`write-confirmation` scenario that omits "must NOT call the execute tool in this
turn" grades a passing H-I-T-L run and a broken one identically.

`input_data` is the fixture contract: the ids and values the run is expected to
resolve. It is how a fixture drift shows up as a failing eval instead of a
quietly wrong demo.

`input_message` is what a practitioner would actually type. Sample question #1
verbatim for the headline scenario.

## Running

`scripts/generate_evals.py` drafts scenarios from a running backend and
`scripts/run_evals.py` runs them. Both carry their usage in their module
docstring — read it rather than guessing flags, and check
`ALL_USE_CASES` in `generate_evals.py` includes the new persona.

Generated scenarios are a first draft. Review every `expected_behavior` against
the canonical demo before committing: the generator writes plausible prose, and
plausible prose grades everything as a pass.

Run them, fix what fails, then set `curated: true`.
