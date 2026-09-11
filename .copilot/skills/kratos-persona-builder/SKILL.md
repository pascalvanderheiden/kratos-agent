---
name: kratos-persona-builder
description: Build a new Kratos use-case persona to the seller-test bar.
disable-model-invocation: true
---

Build a new persona under `use-cases/<name>/` that clears the **seller test**.

A persona is a Microsoft field SE's demo asset. It wins when the customer's own
practitioner watches sample question #1 run and recognises their morning. The
whole build is in service of that one moment.

The repo is the template. The shipped personas already encode the structure;
read the nearest one rather than inventing a shape. This skill carries what
those files do not say out loud.

## The seller test

The bar every persona ships against. Run sample question #1 cold, in a fresh
conversation, and score all six:

| Bar | Passes when |
|---|---|
| **Identity anchor** | The agent never asks who the user is, which company, or what today's date is. It already knows. |
| **Grounded** | Every id in the answer came back from a tool call. Zero invented ids, names, or emails. |
| **Cross-MCP join** | The answer joins two or more systems into a conclusion neither could reach alone. Skills-only personas join two or more skills. |
| **Deliverable** | A real file lands in `/tmp` and **file-sharing** offers it for download. An inline table is a summary, not the deliverable. |
| **H-I-T-L** | Every write stops at a rendered draft and waits for an explicit yes, then reports the receipt id. |
| **Their language** | A practitioner in that vertical recognises the vocabulary, the ids, and the units. |

Six binary bars. A persona that scores five is not shipped.

## Steps

### 1. Claim the slot

Read `use-cases/ROADMAP.md`. It owns what to build next and in what order —
take from the top of the priority queue rather than picking a favourite.

**Done when** you have named the persona, its slot, its vertical, and its MCP
list, and the user has confirmed that choice.

### 2. Pick the donor

From the ROADMAP's shipped table, choose the persona whose **data spine** is
closest — same MCPs, or same skills-only shape. Read its `SYSTEM_PROMPT.md`
end to end and list its `skills/` directory.

**Done when** you can say, for every skill in the donor, whether you copy it
verbatim, adapt it, or drop it — and which skills are net new.

### 3. Cast the identity anchor

Write the cast before writing anything else, because every skill, fixture, and
sample question refers back to it:

- A named human with a role, an employer, and a plant/branch/clinic.
- Their id **in every system** the persona touches (`EMP-*`, `USR-*`, `PRA-*`…).
- Their manager and their directs, with work emails.
- A **fixed** today-date and local timezone. Personas are frozen in time so the
  fixtures always line up.
- Three sample questions, the first of which is the canonical demo.

**Done when** every field above holds a concrete value and no placeholder
survives.

### 4. Lay the data spine

Three routes, by descending cost:

- **Reuse an in-tree mock.** The ROADMAP marks these in bold. Cheapest, and it
  earns a free cross-MCP join against a system another persona already seeded.
- **Skills-only.** Fixtures live in the skill's own `data/` directory. Right for
  a persona whose systems are not worth mocking.
- **New mock.** Follow "Authoring a new mock" in `mocks/README.md`, which owns
  that procedure. Ids must resolve across files, and across the other mocks the
  persona loads — a shared join key is what makes the cross-MCP bar reachable.

**Done when** every id the three sample questions need resolves from a real
tool call, and you have traced the join key end to end by hand.

### 5. Write the skills

One skill per row of the routing table you are about to write, so intent maps
to exactly one destination. Archetypes and conventions: `references/anatomy.md`.

**Done when** every sample question is served end to end, one skill is the
**deliverable**, and **file-sharing** is present.

### 6. Write SYSTEM_PROMPT.md

Frontmatter and section contract: `references/anatomy.md`.

Expand sample question #1 into a numbered, evidence-driven walkthrough under a
`## The canonical demo` heading — the tool calls in order, each one justified by
what the previous call returned. This section is what makes the demo repeatable
rather than lucky.

**Done when** every contract section is present and the canonical demo names a
real tool call at every numbered step.

### 7. Wire it up

- `apm.yml` — copy the donor's, change `name` and `description`.
- `.mcp.json` — one entry per mock, or `{}` for skills-only.
- New mock only: add it to both `src/backend/Dockerfile` and
  `src/hosted-agent/Dockerfile`, and to the table in `mocks/README.md`.
- Move the persona into the ROADMAP's shipped table.
- Add its row to the persona table in `README.md`.
- Leave `curated` out of the frontmatter for now — step 9 earns it.

**Done when** `GET /api/use-cases` returns the persona with a `skillCount`
matching the number of directories under `skills/`.

### 8. Run the seller test

Open a fresh conversation, select the persona, paste sample question #1
verbatim, and score the six bars above.

Score them one at a time against the transcript. A bar is met when you can point
at the evidence — the tool call that produced the id, the file that landed in
`/tmp`, the turn where the draft waited. Any bar you cannot evidence is a fail,
and a fail sends you back to the step that owns it: identity anchor to step 3,
grounded and cross-MCP to step 4, deliverable to step 5, H-I-T-L and their
language to step 6.

**Done when** all six bars are met with evidence, on a cold run you did not
coach.

### 9. Evals, then curate

Write the scenarios and run them: `references/evals.md`.

Set `curated: true` in the frontmatter once they pass. That flag puts the
persona in front of customers in the UI's default view, so it is the last thing
you do, never the first.

**Done when** the scenario suite passes and `curated: true` is set.
