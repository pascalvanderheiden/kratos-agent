# Persona anatomy

The file contract for `use-cases/<name>/`, and the conventions the existing
personas follow without saying so. Read the donor persona alongside this.

## Files

| Path | Required | Notes |
|---|---|---|
| `SYSTEM_PROMPT.md` | yes | Frontmatter + body. The only file the API reads for display metadata. |
| `apm.yml` | yes | Manifest. Copy the donor's; change `name` and `description`. |
| `.mcp.json` | yes | One entry per mock. `{}` when skills-only. |
| `skills/<name>/SKILL.md` | yes | One directory per skill. |
| `skills/<name>/scripts/` | when computing | Python or Node invoked through `code_interpreter`. |
| `skills/<name>/references/` | when citing policy | Static markdown the agent reads on demand. |
| `skills/<name>/assets/` | when rendering | HTML templates for PDF output. |
| `skills/<name>/data/` | when skills-only | JSON fixtures standing in for a system of record. |
| `evals/eval_config.json` | before curating | Copy verbatim from any persona. |
| `evals/scenarios/*.json` | before curating | One file per scenario. |

## SYSTEM_PROMPT.md frontmatter

```yaml
---
name: Plant Floor Supervisor          # display name in the persona picker
description: <one sentence, concrete> # what it does, which systems, the deliverable
sampleQuestions:
  - <the canonical demo>
  - <a second, different shape>
  - <a write, so H-I-T-L is one click away>
curated: true                         # step 9 only
---
```

`description` is the persona's shop window. Name the systems it joins and the
artefact it produces, in the practitioner's vocabulary.

## SYSTEM_PROMPT.md body

The opening paragraph is the **identity anchor**: the named user, their ids in
each system, the employer and site, the fixed date and time, the manager and
directs. Everything downstream leans on it.

Then, in this order:

**`## Default context (do not ask the user for these)`** — restate the anchor as
a lookup table. The anchor has to be reachable at a glance mid-conversation;
a persona that leaves it buried in prose gets asked "which clinician?" on the
first turn, which fails the seller test outright.

**`## Skill routing — MANDATORY`** — a two-column table, user intent to skill.
Every skill in `skills/` appears exactly once. Close it with the grounding rule,
naming the id prefixes that must come from a tool call:

> Do not invent device ids, lot numbers, vendor ids, ticket ids, or employee
> emails. Every `DEV-*`, `PO-*`, `INC-*`, `EMP-*` must come from a tool call.

**`## Mandatory confirmation before any write`** — one row per write surface,
each with its draft tool, its execute tool, and the rule. The contract is
draft → confirm → execute → **report the receipt**. The receipt is what makes the
write believable in a demo: `Created: INC-7012 · assigned Plant Maintenance`.

**`## The canonical demo`** — sample question #1 as numbered tool calls, each
justified by the previous call's output. Evidence-driven: the IoT downtime event
names a `related_po_id`, *which is why* the next call goes to SAP. An agent that
fishes across systems looks like a search engine; an agent that follows a join
key looks like a colleague.

**`## Cross-MCP example — <join>`** — one short section per reusable join, such
as id to name to email, or device to CMDB CI.

**`## Tone & format`** — how the practitioner reads. Lead with the metric. Cite
ids in parentheses, names for the human and codes for traceability. Carry units
and state the rounding. Say which severity buckets come from the tool rather
than being re-derived.

**`## Data disclaimer`** — the simulated-data notice, naming each MCP and the
shared id conventions that link them.

## Skill archetypes

Five shapes cover every skill in the tree.

**MCP wrapper** (`sap-s4`, `workday`, `epic`) — the read interface to one mock.
Carries a tool-routing table, the id conventions, and a `### When NOT to use`
section pointing at its sibling skills. That last section is what keeps the
router from collapsing when two skills touch the same nouns.

**Reference** (`plant-policy-reference`, `sales-playbook-reference`) — a static
policy document under `references/`, plus instructions to read it and **quote the
section number**. Quoting rather than paraphrasing is what makes a compliance
answer defensible on screen.

**Computation** (`oee-analysis`, `variance-analysis`) — a script under
`scripts/` run through `code_interpreter`. Pass tool output straight through as a
JSON blob and let the script parse it; retyping numbers into the prompt is how
demos drift from their fixtures. Document the script's CLI signature rather than
inlining its body.

**Deliverable** (`incident-brief-pdf`, `visit-prep-pack-pdf`) — the artefact the
persona exists to produce. Gathers from the other skills, renders through an
`assets/` HTML template, writes to `/tmp/<thing>-<date>.<ext>`, and states the
path so **file-sharing** picks it up. Ship exactly one.

**Write** (`ticket-actions`, `journal-entry-proposal`) — the H-I-T-L surface.
Draft tool and execute tool are separate, with the confirmation gate between
them and the receipt after.

## SKILL.md shape

```yaml
---
name: oee-analysis     # lowercase-hyphenated, matches the directory
description: <what it does, which files it writes>
enabled: true
---
```

`description` is the router's evidence — write it so the intent it serves is
unmistakable from the description alone. Then `## Instructions`, a `### Workflow`
or `### Tool routing` block, the conventions, and `### When NOT to use`.

Scripts are addressed by their container path: `/app/use-cases/<persona>/skills/<skill>/scripts/<file>`.
