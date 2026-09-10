## Problem Statement

Every agent session on a project loads `CLAUDE.md`/`AGENTS.md`, which in turn points at
`.ai/PROJECT_ARCHITECTURE.md`. That file is therefore paid for in full, on every single session,
before any work starts.

Today that file is produced by the `tdd-red-handoff` plugin's `/init-architecture` command from a
19 KB template, and it grows without a ceiling. Measured on this machine:

- Paddock `.ai/PROJECT_ARCHITECTURE.md`: 76 KB, roughly 19k tokens. Its § Non-Obvious Traps alone
  is 8.1 KB (~2k tokens), § Runtime Wiring 20.3 KB, § Toolchain 4.6 KB.
- KeyFuture `.ai/PROJECT_ARCHITECTURE.md`: 44 KB, roughly 11k tokens.

Most of that volume is knowledge the agent could recover on demand by reading `composer.json`,
`package.json`, or the directory tree. Paying for it up front, every session, buys nothing.

What the agent genuinely cannot recover on demand is the small set of facts that are not written in
any manifest: what a meta-command actually chains and what a red result from it does *not* mean,
which library is absent on purpose and what replaces it, which files are generated and get
overwritten, and the traps that bite an agent doing the obvious thing.

There is no tool that produces that lean file. `/init-architecture` produces the opposite: a
complete, template-shaped document whose value per token falls as the project grows.

## Solution

A user-level skill, `project-architecture`, that writes and maintains
`.ai/PROJECT_ARCHITECTURE.md` in the root of the project it is invoked against, under a hard budget
of ~2k tokens (8,000 characters), containing only knowledge that is **not derivable from the
manifests**.

The skill:

- Mines every source of accumulated project knowledge — the existing file, `CLAUDE.md`/`AGENTS.md`
  at root and in subdirectories, ADRs and `docs/`, plans and post-mortems, `git log` — and keeps
  only what survives an explicit admission rule.
- Writes each finding as a single telegraphic English line carrying its own verifiable anchor (a
  path, a config key, a command name), so that later runs can mechanically re-check it.
- Stamps a provenance marker on line 1 recording its own version, the budget, which sections had no
  material, which sources were explored, and how many candidates were dropped. That marker is what
  makes the next run an incremental update rather than a rewrite.
- Enforces the budget through a fixed eviction order rather than by summarising, so that what
  survives is always the highest-value knowledge, and a budget increase is a deliberate, argued act.

The file it produces is the second level of a two-level system: root `CLAUDE.md`/`AGENTS.md` hold
process (how work is done), `.ai/PROJECT_ARCHITECTURE.md` holds global project facts (how the thing
is built), and `CLAUDE.md` files in subdirectories hold local facts for whoever is already working
there.

This skill is the first act of retiring the `tdd-red-handoff` plugin: ownership of the
`.ai/PROJECT_ARCHITECTURE.md` path moves from the plugin to the skill.

## User Stories

1. As a developer opening any session on a project, I want the always-loaded architecture file to
   cost about 2k tokens instead of 19k, so that the session budget goes to the work rather than to
   background reading.
2. As a developer, I want the always-loaded file to contain only what an agent cannot look up in
   `composer.json` or `package.json`, so that nothing I pay for on every session is redundant.
3. As a developer, I want to invoke the skill explicitly against a project, so that it never starts
   rewriting a project file in the middle of unrelated work.
4. As a developer, I want the skill to refuse to auto-trigger, so that a file that takes user
   confirmations is never regenerated behind my back.
5. As an agent reading the produced file, I want each line in the form `<subject>: <fact> —
   <consequence/action>`, so that I can act on it without decoding prose.
6. As an agent reading the produced file, I want no legend or key at the top, so that no tokens are
   spent on notation in sessions where the skill itself is not loaded.
7. As a developer, I want the file written in telegraphic English, so that it is compact without
   becoming ambiguous, and I can ask for a plain-language translation of any line on demand.
8. As a developer, I want the budget declared in the file itself, so that any future run — or any
   human reading it — knows the ceiling without consulting the skill.
9. As a developer, I want to be warned when the file reaches 90% of its budget, so that I can act
   before the eviction order starts dropping things silently.
10. As a developer, I want the budget to be raisable only in fixed 0.5k increments and only with an
    argued case, so that "it doesn't fit" never quietly becomes "the ceiling is gone".
11. As a developer, I want a budget-increase proposal to list exactly which entries would be
    admitted by the increase, so that I am approving concrete content and not an abstract number.
12. As a developer, I want the first line of the file to identify the skill, the date, the budget,
    the unpopulated sections, the sources explored and the number of dropped candidates, so that the
    next run knows what was already decided.
13. As a developer, I want the absence of that marker to mean "this file was not written by this
    skill", so that the skill can never silently claim authorship of someone else's document.
14. As a developer, I want dropped candidates recorded as a count rather than by name, so that the
    marker does not itself become a second, unbudgeted document.
15. As an agent looking for where to put a new file, I want the shape section first, so that the
    most frequently asked question is answered by the first thing I read.
16. As a developer, I want the shape section to carry identity, layering model, layer-to-directory
    map and deviations, so that project facts currently scattered in `CLAUDE.md` have one home.
17. As an agent about to run a meta-command, I want to know what it actually chains, so that I do
    not misread a composite result.
18. As an agent seeing a red result from a meta-command, I want to know what that red does *not*
    mean, so that I do not chase a failure that is expected.
19. As an agent choosing a library, I want to know which libraries are deliberately absent and what
    is used instead, so that I do not "helpfully" add a dependency that was rejected on purpose.
20. As a developer, I want a deliberate absence recorded only when there is evidence — a doc, an
    ADR, or my own word — so that the file never turns a missing manifest entry into a fake
    decision.
21. As an agent editing a file, I want to know which files are generated and rewritten by tooling,
    so that I do not hand-edit something that will be overwritten.
22. As an agent, I want each trap on one line with a symptom, a prohibition and a pointer, so that I
    can scan the whole set quickly and dig only where I am actually working.
23. As a developer, I want the traps section last, so that the section that grows over time does not
    push the stable sections down the file.
24. As a developer, I want an entry admitted only when an agent would get it wrong *doing the
    default thing* without any reason to go looking, so that the file does not fill with facts that
    are discoverable exactly when they are needed.
25. As a developer, I want a section with no material to be omitted rather than padded, so that the
    file never contains invented content.
26. As a developer, I want an omitted section recorded in the marker, so that a later run knows it
    was considered and found empty rather than skipped.
27. As a developer, I want a fixed eviction order applied when the budget binds, so that what falls
    out is predictable rather than whatever the model felt like cutting.
28. As a developer, I want each entry to carry a verifiable anchor inside its own text, so that
    anchoring costs no extra tokens.
29. As a developer running the skill again later, I want anchors grepped against the current
    codebase, so that entries describing things that no longer exist get flagged.
30. As a developer, I want a stale entry proposed for removal or rewrite rather than removed
    automatically, so that no knowledge disappears without my seeing it.
31. As a developer, I want entries with no possible anchor — historical facts, the reasons behind a
    choice — to survive untouched, so that the update mechanism does not punish knowledge for being
    unverifiable.
32. As a developer running the skill on a project that already has a file with the marker, I want a
    conservative update, so that my previous decisions are preserved.
33. As a developer running the skill on a project whose file has no marker, I want to be warned
    before the shape is replaced, so that the overwrite is never a surprise.
34. As a developer in that case, I want the original archived under `.ai/archive/` with a dated
    name, so that nothing is lost even in a project without git history.
35. As a developer in that case, I want the original *mined* rather than merely archived, so that
    knowledge recoverable nowhere else survives into the new file.
36. As a developer, I want heavy sources — plans, post-mortems, ADRs, `git log` — read by
    subagents, so that the main session context stays clean for the decisions I have to make.
37. As a developer, I want sources consulted in a fixed order ending at the manifests, so that
    accumulated human knowledge is preferred over what the skill could re-derive.
38. As a developer, I want a source that yielded nothing recorded as explored, so that the next run
    does not pay to re-read it unless it has changed.
39. As a developer, I want only genuinely borderline drops put in front of me for validation, so
    that the confirmation step stays short enough to actually read.
40. As a developer, I want the skill to detect facts duplicated between `CLAUDE.md` and the new
    file, so that the same knowledge is not loaded twice per session.
41. As a developer, I want removal from `CLAUDE.md` proposed and executed only on confirmation, so
    that my process document is never edited unilaterally.
42. As a developer, I want removal proposed only for facts actually written into the new file, so
    that an entry dropped for budget is not also deleted from where it currently lives.
43. As a developer whose root `CLAUDE.md`/`AGENTS.md` has no pointer to the file, I want the skill
    to offer to add one, so that a file nothing references is never left behind.
44. As a developer on a project using Laravel Boost, I want the skill to never write to
    `.ai/CLAUDE.md` or `.ai/AGENTS.md`, so that my content is not destroyed by the next
    `composer update`.
45. As a developer with `CLAUDE.md` files in subdirectories, I want the produced file to contain
    only global knowledge, so that the two levels do not overlap.
46. As a developer, I want a stated test for global versus local, so that the split is decidable
    rather than a matter of taste.
47. As a developer, I want local findings listed on screen but not written anywhere, so that the
    skill does not start creating files I did not ask for.
48. As a developer, I want to be asked whether and where to persist local findings, so that the
    destination is my choice.
49. As a developer on a project with the kit installed, I want to be warned that `/verify-kit` will
    go red, so that the failure is expected rather than alarming.
50. As a developer on a project with the kit installed, I want the skill to write the file anyway,
    so that it is usable precisely where the most project knowledge has accumulated.
51. As a developer on a project with no `.ai/` directory, I want it created, so that the skill works
    on projects that never had the kit.
52. As a developer on a project that is not a git repository, I want the `git log` source skipped
    and recorded as unavailable, so that the run completes instead of erroring.
53. As a developer, I want the produced file committed rather than git-ignored, so that it is shared
    knowledge and has a history.
54. As a developer, I want the skill's own procedure separated from the file format and the
    per-stack checklists, so that a run loads only the reference material it actually needs.

## Implementation Decisions

**Identity and packaging**

- New user-level skill named `project-architecture`, living in
  `~/.agents/skills/project-architecture/` with a symlink from `~/.claude/skills/`, matching the
  convention every other skill on this machine follows.
- `disable-model-invocation: true` in the frontmatter. The skill rewrites a project file and asks
  for confirmations; it must never start on its own inside another task.
- `SKILL.md` holds the procedure only. The produced-file format and the per-stack checklists live in
  separate reference files, loaded on demand.
- Promotion to a plugin is deferred. The directory layout should stay trivially liftable into one.
- The skill operates on the project in the current working directory; it takes no path argument.
  Consequence for the collaudo: three separate runs from three directories.

**The produced file**

- Path: `.ai/PROJECT_ARCHITECTURE.md` in the root of the target project. The skill creates `.ai/` if
  it does not exist.
- Audience is the model. Telegraphic English, entry shape `<subject>: <fact> —
  <consequence/action>`, no abbreviations requiring decoding, and no legend — the file is read in
  sessions where the skill is not loaded, so a legend would be a per-session tax.
- Format chosen by measurement on a real trap: prose 75 tokens, telegraphic 40, field notation 30.
  Telegraphic takes ~85% of the saving at zero risk of ambiguity.
- Budget: `max_tokens_k=2`, declared in the file. Measurement is `wc -c` divided by 4. Alarm at 90%
  (7,200 characters).
- A budget increase happens only in 0.5 increments (2,000 characters), and only when, after the
  eviction order has been applied, entries of priority 1–3 are still excluded. The proposal must
  enumerate which entries the increase would admit.
- Line 1 is the provenance marker:

  ```
  <!-- project-architecture v1 | updated <date> | max_tokens_k=2 | not-populated: … | sources-explored: … | dropped: N -->
  ```

  No marker means the file is foreign. Drops are recorded as a count, never by name.
- Section skeleton and indicative spend — the hard constraint is the total, not the per-section
  share:

  ```
  ## Shape                 ~200 tok  identity, layering model, layer→dir map, deviations
  ## Commands              ~250 tok  what each meta-command chains, and what a red does NOT mean
  ## Deliberately absent   ~150 tok  libraries absent by choice + the alternative in use
  ## Runtime wiring        ~400 tok  generated files, plugins that rewrite, disabled routing
  ## Traps                 ~900 tok  one line per trap, verifiable anchor included
  ```

- `Shape` first because it answers the most frequent question ("where does this file go"). `Traps`
  last because it is the section that grows. The `Shape` heading is full — identity, layering, map,
  deviations — because the `CLAUDE.md` files are all going to be revised and project facts migrate
  here.
- Reference form for the entry style already exists in production: `Paddock/CLAUDE.md`, § Non-Obvious
  Traps — 13 entries, one line each, symptom + prohibition + pointer to the anatomy, 3,092
  characters (~770 tokens). Hand-written, already two-level. Use it as the format model.

**Content rules**

- Admission rule: an entry qualifies only if an agent gets it wrong *doing the default thing*,
  without having any reason to go looking. A `composer test` meta-command qualifies (you run it
  regardless). Middleware ordering does not (you only reach it while writing a guard, and at that
  point you have a reason to search).
- Eviction order, from the bottom up: 5 wiring / 4 noisy-failure traps / 3 silent-failure traps /
  2 deliberate omissions / 1 command semantics. Priority 5 goes first, priority 1 last.
- Do not pad. If a section has no material at T0 it is omitted and recorded in the marker. Weak
  traps get added at Tn; they are not invented at T0.
- Deliberate omissions require evidence — a doc, an ADR, or the user's own statement. They are never
  inferred from absence in `package.json`. Each carries a reason and the alternative in use: "zod:
  absent by choice, boundary validation lives in Form Requests; do not add it", never "zod NOT
  INSTALLED".
- Atomic granularity: one entry, one line, with its source, so that merging at Tn is mechanical.
- Global versus local test — radius of effect. Global if an agent trips on it starting from anywhere
  in the repo, or if the effect crosses more than one directory. Local if it only affects someone
  already working there. Local findings are **listed only**; the skill asks whether and where to
  persist them (a file in root, a Notion note, another destination reachable by the model). It does
  not write them and does not propose a destination itself.

**Execution cycle**

1. Detect the existing file. Marker present → conservative update. Marker absent → warn that the
   shape will be replaced, archive the original to `.ai/archive/PROJECT_ARCHITECTURE.<date>.md`, and
   **mine its content**: information recoverable nowhere else must be preserved.
2. Mixed excavation. In-session: the file at the same path (richest source, must be understood) and
   the `CLAUDE.md`/`AGENTS.md` files. Delegated to subagents to keep the main context clean:
   `.ai/plans/` and post-mortems, ADRs and `docs/`, `git log` (reverts, repeated fixes on the same
   spot). Source order: existing file → root and subdirectory `CLAUDE.md`/`AGENTS.md` → ADRs/docs →
   plans → `git log` → manifests and scripts (the last only to understand what is inside the
   commands). A source that yields nothing is recorded in the marker as explored, and is not
   reopened on the next run unless it has changed.
3. Write the file and ask for validation. On screen, list only the drop decisions that are genuinely
   borderline; obvious drops are not submitted.
4. Handle duplication with `CLAUDE.md`. Boundary rule: `CLAUDE.md` = process (how work is done),
   `.ai/PROJECT_ARCHITECTURE.md` = facts (how the thing is built). The skill copies the facts into
   its own file and **proposes** removal from `CLAUDE.md` — only of entries actually written, never
   of entries dropped for budget. Executes on confirmation.
5. Hook. If the root `CLAUDE.md`/`AGENTS.md` has no pointer: "I did not find a hook to this file,
   shall I add one?". Never `.ai/CLAUDE.md` or `.ai/AGENTS.md` — on Paddock those are generated by
   Laravel Boost via `config/boost.php` → `agents.*.guidelines_path` and rewritten on every
   `composer update`. Proposed line:

   ```
   Project facts (commands, wiring, traps, deliberate absences): read .ai/PROJECT_ARCHITECTURE.md in full before planning or editing.
   ```

6. Projects with the kit active. Warn that `/verify-kit` will fail while the plugin remains
   installed. Write anyway: refusing would make the skill useless exactly where the most context has
   accumulated.

**Update at Tn**

- Every entry carries a verifiable anchor inside its own text — a path, a config key, a command name
  (`DB_HOST=paddock-mysql`, `resources/js/actions/`, `config/boost.php`). Zero additional cost: the
  anchor *is* the entry text, not an extra field.
- At Tn the skill greps the anchors. Gone or changed → the entry is **suspect**, proposed for removal
  or rewrite. **Never removed automatically.**
- Non-anchorable entries (historical facts, the reasons behind a choice) are not verifiable and
  survive until the user touches them.

**Known regression, accepted**

The plugin's `bin/verify-kit.sh` reads the live file and checks mechanically: the seven command
names (`test`, `test (focused)`, `typecheck`, `lint`, `format`, `format:check`, `coverage`) present
verbatim in the § Toolchain table; the `**Project floor:**` line matching the one in `CLAUDE.md`; no
model name outside § Model Roster. The lean file has none of that structure, so `/verify-kit` goes
red on kit projects while the plugin stays installed. Decision: warn and proceed.

**Stated assumptions**

1. The produced file is committed, not ignored: it is shared knowledge, and without git history the
   archive is the only safety net.
2. The file's language is English (a consequence of the chosen format).
3. If `.ai/` does not exist, the skill creates it.
4. If the project is not a git repository, the `git log` source is skipped and the marker records it
   as unavailable.

## Testing Decisions

**What makes a good test here.** The skill is a Markdown procedure, not code: there is no function
to call. A good test asserts only **externally observable outcomes** — what ends up on disk after a
run — and never the skill's internal procedure, the order in which it read sources, or the wording
of its on-screen prompts. Asserting "the file is under 8,000 characters" is a test; asserting "the
skill read the ADRs before the plans" is testing the implementation.

**The seam.** One seam, and it already exists: **the produced `.ai/PROJECT_ARCHITECTURE.md`, marker
line included**. Everything worth checking is observable there:

- total size within budget (`wc -c` ≤ 8,000; alarm threshold 7,200 respected);
- marker present on line 1, with all its fields populated;
- only the defined sections present, in the defined order, with empty ones omitted rather than
  padded;
- one line per entry, each carrying a greppable anchor;
- `.ai/CLAUDE.md` and `.ai/AGENTS.md` untouched (compare before/after);
- `.ai/archive/PROJECT_ARCHITECTURE.<date>.md` written whenever a file without a marker was
  replaced;
- root `CLAUDE.md`/`AGENTS.md` modified only where the user confirmed it.

No new seam is introduced. In particular, **no shell validator script is added**: it would pull the
skill's structure toward the plugin shape being retired, and mechanical coupling to this file's
structure is exactly what breaks `/verify-kit` today.

**Cases run through the seam.** Three real projects, in order, chosen for what each one proves:

1. **KeyFuture** — 44 KB existing file, same stack as Paddock, half the volume. Exercises the full
   cycle including mining a foreign file.
2. **Paddock** — 76 KB, roughly 30 candidate entries, a `CLAUDE.md` section to prune. The worst
   case: it is where the budget actually binds and the eviction order gets exercised.
3. **MailRules** — Python, no `CLAUDE.md`, no kit, and a 13 KB `DECISIONI.md`. Proves the skill does
   not pad sections just to fill them.

**Pass criteria.** The mechanical checks above are necessary, not sufficient — they cannot tell
whether the right 2k tokens survived out of 19k.

- **KeyFuture** — PASS when every entry is traceable to the archived 44 KB original or another
  named source, and the marker's `dropped:` count plus the surviving entries account for the
  candidates found.
- **Paddock** — the expected-output set already exists: § Non-Obvious Traps in `CLAUDE.md`, 13
  entries at 3,092 characters (~770 tokens), which fits inside the ~900-token Traps budget. PASS
  when the produced § Traps carries those 13, or names each one it dropped together with the
  eviction priority that dropped it. Those 13 consume ~85% of the Traps budget before the other
  ~17 candidates are considered, so eviction is expected here — an *unexplained* drop is not.
- **MailRules** — PASS when sections with no material are absent and named in the marker's
  `not-populated:` field, and the 13 KB `DECISIONI.md` was read as a source rather than
  transcribed.

**Reversibility.** The KeyFuture and Paddock runs overwrite live 44 KB and 76 KB files. Before each
run, the write must be reversible: a git branch, or confirmation that the
`.ai/archive/PROJECT_ARCHITECTURE.<date>.md` step landed. MailRules has no `.ai/` and no existing
architecture file, so its run is a create, not an overwrite: reversing it means deleting what was
written.

**Prior art.** `bin/verify-kit.sh` in the `tdd-red-handoff` repo is the existing example of
mechanical assertions over this exact file, and its tests live in `tests/verify-kit/`. It is prior
art to read for what to check, and a deliberate non-model for how to package the checking.

## Out of Scope

- **A validator script.** No `bin/` executable that checks the produced file. Decided explicitly;
  reopen only if manual verification proves unworkable across the three collaudo projects.
- **Promotion to a plugin.** The skill ships user-level. The layout stays liftable, but the lift is
  not part of this work.
- **A second, "deep" architecture file.** Superseded: the second level is the `CLAUDE.md` files in
  subdirectories, which the user maintains.
- **Writing or proposing local-knowledge destinations.** The skill lists local findings and asks; it
  does not create subdirectory `CLAUDE.md` files.
- **Retiring the `tdd-red-handoff` plugin.** This skill is the first act of it, not the whole of it.
  Uninstalling the plugin, removing `/init-architecture`, `/update-kit` and `/verify-kit`, and
  cleaning `.ai/kit.json` out of the projects are separate work.
- **Fixing `/verify-kit`.** It goes red on kit projects and stays red. No change to
  `bin/verify-kit.sh` is in scope.
- **The ADRs explaining that regression.** A sensible ADR exists — why `/verify-kit` turns red and
  why the kit is being retired — but it belongs in the Paddock and KeyFuture repos, not here.
  Follow-up work.
- **`CONTEXT.md` or an ADR for the skill itself.** The skill's own vocabulary (marker, anchor,
  radius of effect, eviction order) belongs in its reference files.
- **Migrating every project on the machine.** Three projects are the collaudo. AgentsConfigTemplate
  and any other project are separate decisions.
- **Rewriting the `CLAUDE.md` files.** The skill proposes removal of duplicated facts and executes
  on confirmation; the broader `CLAUDE.md` revision the user has in mind is its own task.

## Further Notes

- The budget is not a target to hit. It is a constraint that forces an eviction rule. At current
  fidelity the four wanted categories come to ~33 KB (~8k tokens) — four times the ceiling — so the
  eviction order will be exercised on the first real project, not hypothetically.
- The format decision was made by measurement, not preference: 75 → 40 → 30 tokens for prose,
  telegraphic and field notation on the same real trap. The middle option was chosen deliberately.
- Ownership of the `.ai/PROJECT_ARCHITECTURE.md` path legitimately transfers from the plugin to the
  skill. Both will claim it while the plugin remains installed; the plugin is the one on its way
  out.
- The projects on this machine at the time of writing: Paddock and KeyFuture have the kit active
  (`.ai/kit.json`), AgentsConfigTemplate is the template for new projects, MailRules is Python with
  no `CLAUDE.md`, Skills is the skills repo.
- Source of these decisions: grilling session of 2026-09-10, recorded in
  `project-architecture-decisions.md`.
