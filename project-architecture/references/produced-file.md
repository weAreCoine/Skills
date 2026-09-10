# The produced file

Contract for `.ai/PROJECT_ARCHITECTURE.md`: where it goes, how it reads, what is allowed in.

## Location

`.ai/PROJECT_ARCHITECTURE.md`, in the root of the target project. Create `.ai/` if it is missing.

Never write `.ai/CLAUDE.md` or `.ai/AGENTS.md`. On some stacks those are generated — Laravel Boost
writes them from `config/boost.php` → `agents.*.guidelines_path` and rewrites them on every
`composer update`. Anything put there is destroyed.

The file is committed, not git-ignored: it is shared knowledge, and its history is the safety net.

## Audience, language, entry shape

The reader is the model, not a human. Telegraphic English. One entry, one line:

```
<subject>: <fact> — <consequence/action>
```

No legend, no key, no notation to decode. The file is read in sessions where this skill is not
loaded, so a legend would be a tax paid on every session.

Measured on a real trap: prose 75 tokens, telegraphic 40, field notation 30. Telegraphic takes ~85%
of the saving at zero risk of ambiguity. Any line can be translated into plain language for a human
on request; the file itself stays telegraphic.

## Budget

Declared in the file as `max_tokens_k=2`. Measure with `wc -c`; tokens ≈ characters / 4.

- Hard ceiling: **8,000 characters**.
- Alarm: **7,200 characters** (90%). Report it; the file still ships.

The budget is a constraint, not a target. A short file is a good file — padding a section to reach
the ceiling is the failure this file exists to avoid.

## Line 1 — the provenance marker

```
<!-- project-architecture v1 | updated YYYY-MM-DD | max_tokens_k=2 | not-populated: <sections> | sources-explored: <sources> | dropped: N -->
```

| Field | Meaning |
|---|---|
| `project-architecture v1` | Authorship and format version. |
| `updated` | ISO date of the run that wrote the file. |
| `max_tokens_k` | The budget in force, declared so that a later run knows the ceiling without loading this skill. |
| `not-populated` | Sections omitted for lack of material, comma separated. `none` when all sections are present. |
| `sources-explored` | Every source opened, comma separated, named as a path a later run can reopen or diff — `DECISIONI.md`, not `docs`. Add a status where one is needed: `absent`, `unavailable`, `nothing-found`. Includes the sources that yielded nothing. |
| `dropped` | Number of candidates considered and rejected. A count, never names — names would turn the marker into a second, unbudgeted document. |

Every field is always populated. **No marker means the file was not written by this skill**: that is
the only authorship test, and there is no other way to claim a file.

## Sections

Fixed order. The indicative spend is guidance; the hard constraint is the total.

| Section | ~tokens | Holds |
|---|---|---|
| `## Shape` | 200 | Identity, layering model, layer→directory map, deviations from it. |
| `## Commands` | 250 | What each meta-command actually chains, and what a red result does NOT mean. |
| `## Deliberately absent` | 150 | Libraries absent by choice, plus the alternative in use. |
| `## Runtime wiring` | 400 | Generated files, tooling that rewrites files, disabled routing. |
| `## Traps` | 900 | One line per trap: symptom, prohibition, anchor. |

`Shape` first: it answers the most frequent question, "where does this file go". `Traps` last: it is
the section that grows, and it must not push the stable sections down the file.

A section with no material is **omitted** — not written empty, not padded — and named in the
marker's `not-populated:` field, so a later run knows it was considered and found empty rather than
skipped. Weak traps are added on a later run; they are not invented now.

## Admission rule

An entry qualifies only if an agent gets it wrong **doing the default thing**, without having any
reason to go looking.

- Qualifies: a `composer test` meta-command that also runs the linter. You run it regardless, and
  you misread its red.
- Does not qualify: middleware ordering. You only reach it while writing a guard, and at that point
  you have a reason to search.

Anything stated in a manifest or visible in the directory tree is out by construction: the agent
reads it exactly when it needs it, and paying for it every session buys nothing.

One exception, and only one. `## Shape` carries the layering **model**: which layer may call which,
where a file of a given kind belongs, and where the project knowingly deviates from its own rule.
The tree shows what exists; it does not show which rule governs where the *next* file goes, and an
agent placing a file gets that wrong by default. A plain listing of the directories is still
inadmissible — write the rule, not the inventory.

## Anchors

Every entry carries a verifiable anchor inside its own text — a path, a config key, a command name
(`DB_HOST=paddock-mysql`, `resources/js/actions/`, `config/boost.php`). The anchor **is** the entry
text, not an extra field, so it costs no tokens. Later runs grep those anchors to find entries
describing things that no longer exist.

Entries with no possible anchor — historical facts, the reason behind a choice — are admissible.
They are simply not verifiable, and survive until the user touches them.

## Deliberate omissions need evidence

A library is "absent by choice" only when there is evidence: a document, an ADR, or the user saying
so. Never inferred from its absence in a manifest — absence in `package.json` is not a decision.

Each one carries its reason and the alternative in use:

```
zod: absent by choice, boundary validation lives in Form Requests — do not add it
```

Never `zod NOT INSTALLED`.

## Global vs local — radius of effect

- **Global**, so it belongs in this file: an agent trips on it starting from anywhere in the repo,
  or the effect crosses more than one directory.
- **Local**, so it does not: it only affects someone already working in that directory.

Local findings are listed on screen and written nowhere. Ask whether and where to persist them, and
propose no destination — a subdirectory `CLAUDE.md`, a note elsewhere, or nothing at all is the
user's choice.

## Granularity

One entry, one line, one fact, carrying its own anchor. Atomic entries make the merge on a later
run mechanical instead of a rewrite.

## Worked example

Shape only — the entries below are illustrative, not a template to copy:

```markdown
<!-- project-architecture v1 | updated 2026-09-10 | max_tokens_k=2 | not-populated: Deliberately absent | sources-explored: existing-file:absent, CLAUDE.md, docs/adr/:nothing-found, git-log | dropped: 14 -->

## Shape
Identity: Laravel monolith serving an Inertia SPA — one deployable, no separate frontend build target.
Layering: HTTP → app/Actions/ → app/Models/ — controllers hold no business logic, put it in an Action.

## Commands
composer test: chains Pest + PHPStan + Pint --test — a red does NOT mean a test failed, read which stage stopped.

## Runtime wiring
resources/js/actions/: generated by laravel/wayfinder on composer update — hand edits are lost, change the PHP route instead.

## Traps
Queue driver: .env sets QUEUE_CONNECTION=sync in local — dispatched jobs run inline, so a job that "works locally" is untested against the queue.
```
