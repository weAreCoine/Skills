---
name: project-architecture
description: "Write .ai/PROJECT_ARCHITECTURE.md for the project in the current directory — a ~2k-token file holding only the project facts an agent cannot recover from the manifests."
disable-model-invocation: true
---

Once a root `CLAUDE.md`/`AGENTS.md` pointer exists, `.ai/PROJECT_ARCHITECTURE.md` is loaded by every
agent session on the project and paid for in full before any work starts. So it holds only what an
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

For either continuing path, take a pre-run snapshot before mining:

- Record whether root `CLAUDE.md` and `AGENTS.md` exist, whether either is a symlink or both resolve
  to the same file, and their checksums. These are the only files this skill may later offer to edit.
- Treat `.ai/CLAUDE.md`, `.ai/AGENTS.md`, and `bin/verify-kit.sh` as the protected-file set. Record
  whether each exists and, when present, its checksum. Protected files are never edited.
- If `.ai/kit.json` exists, warn that `/verify-kit` will fail while the `tdd-red-handoff` plugin
  remains installed. This warning is informational: continue writing the architecture file. Never
  change `bin/verify-kit.sh` to make the legacy check pass.

### Foreign-file path

Warn now, before any mutation: the file is foreign, its shape will be replaced, and the exact
original will be archived at `.ai/archive/PROJECT_ARCHITECTURE.<YYYY-MM-DD>.md`.

Make the run reversible before mining or replacement:

1. Record the foreign file's checksum.
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
2. Root `CLAUDE.md` and `AGENTS.md`. Classify their content by the boundary `process = how work is
   done`, `project fact = how this project is built`. For every project-fact candidate, record its
   exact source span so a later removal proposal can show a reviewable diff. When the files resolve
   to the same underlying file, mine that content once and record both paths.
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
its final entry text if kept, and its decision: admitted, local, or dropped with reason. For every
admitted entry also record the eviction priority defined in `references/produced-file.md`, or
`not-evictable` for Shape. For a candidate mined from a root process document, retain its exact
source span and mark it as a possible duplicate. Duplicate evidence from a later source corroborates
the existing candidate; it is not a new candidate.

Local candidates, rejected candidates and later budget evictions all count as dropped. Before every
write, verify:

```
candidate count = final surviving entry count + dropped count
```

The right-hand `dropped count` is the marker's `dropped:` field. Every surviving entry must map to
one named source in the ledger.

## 5. Render a draft

Render the complete candidate file without replacing `.ai/PROJECT_ARCHITECTURE.md`: marker on line
1, the defined sections in the defined order, sections with no material omitted and named in the
marker. The marker and separators are part of the measured draft.

## 6. Enforce the budget and write

Run `wc -c` on the draft. If it is above the ceiling, apply the eviction procedure in
`references/produced-file.md`: update the ledger and `dropped:` count after each whole-entry
eviction, render again, and remeasure. The draft must be within budget before the live file changes.

If no eligible entry remains and the draft is still over the ceiling, STOP. Report the failed run
and leave the live architecture file unchanged; compression is not a fallback.

Before replacing a foreign file, verify the dated archive's checksum. If it differs from the
recorded foreign-file checksum, the archive is not a recovery source: report the failed run and make
no further writes. Restore the live architecture file only from an archive whose checksum still
matches the recorded original.

On both create and foreign-file paths, compare the protected-file set's current existence and
checksums with the pre-run snapshot. A mismatch is a failed run and stops the write. Also verify the
root `CLAUDE.md` and `AGENTS.md` snapshot: neither may change before the explicit integration
confirmations in step 7.

Create `.ai/` if it does not exist, replace `.ai/PROJECT_ARCHITECTURE.md` with the passing draft,
then run `wc -c` on the live file and verify the ceiling again. Verify again that the protected-file
set still matches the pre-run snapshot.

| Result | What to do |
|---|---|
| Under the alarm threshold | Fine. Ship it. |
| At or above the alarm, at or under the ceiling | Ship it and warn that further growth will cause more eviction. |
| Above the ceiling | Failed enforcement. Restore the foreign file only from its verified archive; on the create path, remove the invalid output. |

If any budget-evicted entry has priority 1–3, prepare the one-increment proposal defined in
`references/produced-file.md`. Do not change the passing file or its marker until the user approves
the enumerated entries.

## 7. Reconcile the surrounding project

Only facts that survived into `.ai/PROJECT_ARCHITECTURE.md` are eligible for deduplication. A
candidate rejected by admission, classified local, or dropped by budget stays in its source.

1. From the ledger, list each surviving entry whose fact also remains in root `CLAUDE.md` or
   `AGENTS.md`. Pair the produced entry with the exact root-document span; this is the duplication
   report. Do not treat a pointer or process instruction as a duplicate project fact.
2. Prepare an exact removal diff containing only those listed spans and propose it to the user. Do
   not edit either root document before confirmation. Apply only the hunks the user explicitly
   confirms, and account for symlinks or shared file identity so one confirmation never causes an
   undisclosed second edit.
3. After any confirmed removals, inspect both root documents for a pointer to
   `.ai/PROJECT_ARCHITECTURE.md`. If neither contains one, ask exactly: "I did not find a hook to
   this file, shall I add one?" Show the target file and this proposed line:

   ```text
   Project facts (commands, wiring, traps, deliberate absences): read .ai/PROJECT_ARCHITECTURE.md in full before planning or editing.
   ```

   Add it only after confirmation. If both root documents exist, disclose whether they are distinct
   files and which one or ones the proposal would change.
4. Compare the final root-document diff with the approved removal and hook hunks. If anything else
   changed, stop and report the unexpected diff; make no further root-document edits.
5. Verify the protected-file set once more against the pre-run snapshot. Absence must also remain
   unchanged.

## 8. Report and validate

On screen, and nowhere else:

- The path written, its size in characters, and its marker line.
- On the foreign-file path: the archive path, proof that it matched the original before replacement,
  and a source mapping for every surviving entry from the candidate ledger.
- Candidate accounting: total candidates, surviving entries, and dropped count.
- Every budget eviction, with the entry text and its priority. These details stay on screen; the
  marker carries only the count.
- **Borderline** drop decisions only. Obvious drops are not submitted for validation — the point of
  the list is that it stays short enough to actually read.
- **Local findings**, in full. They are written nowhere. Ask whether and where to persist them, and
  propose no destination.
- Every root-document duplicate found, whether its removal was proposed, and which confirmed hunks
  were applied. List budget-dropped entries separately; none may appear in the removal proposal.
- Whether a hook was already present, declined, or added, and the exact root-document paths changed.
- The result for each path in the final protected-file comparison. When `.ai/kit.json` was present,
  repeat that `/verify-kit` is expected to fail while the `tdd-red-handoff` plugin remains installed
  and that the architecture file was written anyway.
- How to reverse the run: on the create path, delete the file and the `.ai/` directory too if this
  run created it; on the foreign-file path, copy the dated archive back over the generated file.
