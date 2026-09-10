---
name: project-architecture
description: "Write .ai/PROJECT_ARCHITECTURE.md for the project in the current directory — a ~2k-token file holding only the project facts an agent cannot recover from the manifests."
disable-model-invocation: true
---

`.ai/PROJECT_ARCHITECTURE.md` is loaded by every agent session on the project, through the pointer in
`CLAUDE.md`/`AGENTS.md`. It is paid for in full before any work starts. So it holds only what an
agent **cannot** recover on demand from `composer.json`, `package.json`, `pyproject.toml` or the
directory tree, and it stays inside the budget `references/produced-file.md` declares.

This skill writes that file for the project in the **current working directory**. It takes no path
argument: to run it against another project, change directory first.

Read `references/produced-file.md` before writing anything. It is the contract for the file's
format, its marker, and the rule that decides what gets in. This file says *when* to apply those
rules; it does not restate them, so the budget, the section list and the admission rule each have
exactly one home.

## 1. Establish the target state

Read the first line of `.ai/PROJECT_ARCHITECTURE.md` in the current directory.

| State | What to do |
|---|---|
| File absent | Continue. This is the create path, the one this version implements. |
| Line 1 carries the `project-architecture` marker | STOP. This is an incremental update. Not implemented in this version: report it and change nothing. |
| File present, line 1 has no marker | Continue on the foreign-file path below. |

### Foreign-file path

Warn now, before any mutation: the file is foreign, its shape will be replaced, and the exact
original will be archived at `.ai/archive/PROJECT_ARCHITECTURE.<YYYY-MM-DD>.md`.

Make the run reversible before mining or replacement:

1. Record the foreign file's checksum. Also record whether `.ai/CLAUDE.md` and `.ai/AGENTS.md`
   exist and, when present, their checksums.
2. Create `.ai/archive/` and copy the foreign file to the dated archive path. If that destination
   already exists with different bytes, STOP and report the collision; do not overwrite either
   file.
3. Verify the live file and archive are byte-identical with `cmp -s`. A failed comparison stops the
   run. The live file is not replaced until step 5.

The verified archive is the source mined during this run and the recovery copy if the generated
file is rejected. A marker-bearing file is never archived or replaced by this version.

## 2. Identify the stack

Pick the stack from the manifest present at root: `pyproject.toml`, `requirements.txt` or loose
`*.py` → Python; `composer.json` → PHP; `package.json` → Node.

Load `references/stacks/<stack>.md` if it exists. It is a checklist of what to look at on that
stack, not a source of facts. If there is no checklist for the stack at hand, proceed without one
and say so in the final report.

## 3. Excavate the sources

Fixed order. Accumulated human knowledge first, re-derivable material last.

1. `.ai/PROJECT_ARCHITECTURE.md` — the richest source when it exists. On the foreign-file path,
   read the verified dated archive in full and mine it before any other source; identify it by its
   archive path in the candidate ledger and marker. On the create path it is absent; record it as
   such.
2. Root `CLAUDE.md` and `AGENTS.md`.
3. `CLAUDE.md` and `AGENTS.md` in subdirectories. These carry **local** knowledge. Read them to
   know what is already covered down there; do not copy local facts up into this file.
4. ADRs and project documents, including `docs/`, decision logs and `README.md`.
5. Plans and post-mortems, including `.ai/plans/`.
6. `git log` — a bounded slice (`git log --oneline -200`) looking for reverts and repeated fixes on
   the same spot.
7. Manifests and scripts. Last, and only to learn what a meta-command actually chains. A fact that
   is merely *stated* in a manifest is inadmissible by construction — the agent can read it when it
   needs it.

Steps 4–6 are delegated excavation. Inventory their exact paths without opening their contents,
then dispatch one read-only subagent for each available group: ADRs/docs, plans/post-mortems, and
git history. If subagents are unavailable, STOP before replacing the live file and report that this
run cannot preserve the required context boundary. Do not read those heavy sources inline.

Each subagent returns:

- every exact path or git range it inspected, including sources that yielded no candidate;
- candidate facts, one per item, with proposed entry text, section, source path or commit, anchor,
  and global/local classification;
- local findings separately;
- no file changes.

For `git log`, check availability before dispatch with `git rev-parse --git-dir`. No repository or
no usable history means no subagent: record `git-log:unavailable` and continue.

Subagents may finish in any order. Merge their results only in the numbered source order above;
when sources disagree, the earlier accumulated-human source wins and the later source is only
corroboration. The main session applies admission and makes every keep/drop decision.

Never write to `.ai/CLAUDE.md` or `.ai/AGENTS.md` — `references/produced-file.md` says why. This
skill only ever reads them.

Record every source in the marker's `sources-explored:`, **including the ones that yielded nothing**.
Name each one so a later run can reopen it or diff it: file paths, not categories; append
`:nothing-found`, `:absent`, or `:unavailable` when applicable. A source that produced nothing is
itself a fact the incremental-update path can later use.

## 4. Select what gets written

Apply, in this order, the rules in `references/produced-file.md`:

1. The admission rule — an entry qualifies only if an agent gets it wrong doing the default thing.
2. The global-vs-local test — radius of effect. Local findings do not go in the file.
3. The evidence rule for deliberate omissions.
4. The anchor rule — every entry carries its own verifiable anchor.

Maintain one candidate ledger across all sources. For every candidate record its earliest source,
its final entry text if kept, and its decision: surviving, local, or dropped with reason. Duplicate
evidence from a later source corroborates the existing candidate; it is not a new candidate.

Local candidates and all other rejected candidates count as dropped. Before writing, verify:

```
candidate count = surviving entry count + dropped count
```

The right-hand `dropped count` is the marker's `dropped:` field. Every surviving entry must map to
one named source in the ledger.

## 5. Write the file

Create `.ai/` if it does not exist. Only now replace `.ai/PROJECT_ARCHITECTURE.md`, writing it
exactly as `references/produced-file.md` specifies: marker on line 1, the defined sections in the
defined order, sections with no material omitted and named in the marker.

## 6. Measure

Run `wc -c` on the produced file and compare it against the ceiling and the alarm threshold in
`references/produced-file.md`.

On the foreign-file path, verify the dated archive's checksum first. If it differs from the recorded
foreign-file checksum, the archive is not a recovery source: report the failed run and make no
further writes. Then compare the current `.ai/CLAUDE.md` and `.ai/AGENTS.md` state and checksums with
the snapshots from step 1; a mismatch is a failed run and stops further writes. Restore the live
architecture file only from an archive whose checksum still matches the recorded original.

| Result | What to do |
|---|---|
| Under the alarm threshold | Fine. Ship it. |
| At or above the alarm, under the ceiling | Ship it, and report the alarm: the next run will have to evict. |
| Above the ceiling | Do not ship it as is. Enforcing the budget by eviction is not implemented in this version. Report the overflow, give the size of each section, and **recommend** which entries to cut and why — one concrete proposal with its cost, not an open question. Write the file once the user has chosen. |

## 7. Report and validate

On screen, and nowhere else:

- The path written, its size in characters, and its marker line.
- On the foreign-file path: the archive path, proof that it matched the original before replacement,
  and a source mapping for every surviving entry from the candidate ledger.
- Candidate accounting: total candidates, surviving entries, and dropped count.
- **Borderline** drop decisions only. Obvious drops are not submitted for validation — the point of
  the list is that it stays short enough to actually read.
- **Local findings**, in full. They are written nowhere. Ask whether and where to persist them, and
  propose no destination.
- Whether anything in the project points at the file. If nothing does, say so plainly. Adding the
  pointer is not part of this version.
- How to reverse the run: on the create path, delete the file and the `.ai/` directory too if this
  run created it; on the foreign-file path, copy the dated archive back over the generated file.
