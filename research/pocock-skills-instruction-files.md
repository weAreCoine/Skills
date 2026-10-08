# Cosa scrivono e cosa leggono le skill di Matt Pocock nei file del progetto

Ricerca per COINE-140 (mappa COINE-138). Data: 2026-10-08.

## Fonti e perimetro

- **Skill installate**: i symlink in `/Users/luca/.claude/skills/` che puntano dentro `/Users/luca/Projects/my-pocock-skills/skills/`. Sono 34: tutto `engineering/` (20), tutto `productivity/` (7), tutto `in-progress/` (8, incluse `serial-implement` e `setup-ts-deep-modules`). Le skill di `misc/` (`git-guardrails-claude-code`, `migrate-to-shoehorn`, `scaffold-exercises`, `setup-pre-commit`) sono installate da `~/.agents/skills/`, non da questo repo, e nessuna tocca i file elencati sotto.
- **Versione installata**: i symlink puntano al working tree, quindi la versione installata è il branch checked out in `my-pocock-skills`: `feat/italian-conversation-language` @ `99405cb` (pushato su `origin`, **non ancora mergiato** in `origin/main` @ `cce0aeb`). Il working tree non ha modifiche sotto `skills/` (`git status`: solo `scripts/link-skills.sh` modificato).
- **Upstream**: remote `upstream` = `git@github.com:mattpocock/skills.git`, ultimo fetch locale 2026-10-07 (`.git/FETCH_HEAD`). `upstream/main` @ `f3fc563`. Non ho fatto fetch.
- **Esempio reale**: `/Users/luca/Projects/Paddock/CLAUDE.md` (`AGENTS.md` è un symlink a `CLAUDE.md`) e `/Users/luca/Projects/Paddock/docs/agents/`.
- Grep su ogni `SKILL.md` e file di riferimento per: `CLAUDE.md`, `AGENTS.md`, `docs/agents`, `CONTEXT.md`, `GLOSSARY`, `GLOSSARY-MAP`, `docs/adr`, `docs/index.md`, `PROJECT_TRACKING`, `.ai/`; più i riferimenti indiretti ("the issue tracker should have been provided to you", "the tracker doc", "the project's domain glossary").

Tutti i percorsi `skills/...` sotto sono relativi a `/Users/luca/Projects/my-pocock-skills/`.

## Tabella: file → chi scrive → chi legge

"Ogni sessione" = il file è caricato dall'harness nel contesto di ogni sessione senza che una skill lo chieda.

| File (nel repo del progetto) | Skill che lo scrivono | Skill che lo leggono (condizione) | Ogni sessione? | Sorgente (path:riga) |
|---|---|---|---|---|
| `CLAUDE.md` / `AGENTS.md` (root) | `setup-matt-pocock-skills`: aggiunge o aggiorna in place il blocco `## Agent skills`. Sceglie `CLAUDE.md` se esiste, altrimenti `AGENTS.md`; se nessuno dei due esiste **chiede** quale creare. Non crea mai l'altro. | `setup-matt-pocock-skills` (esplorazione, sempre); `serial-implement` implementer (comandi di test e "si applica", sempre); `retro` (li valuta come candidati, sempre); `code-review` (indiretto: "every file that documents how code should be written") | **Sì** (dall'harness: Claude Code legge `CLAUDE.md`, Codex legge `AGENTS.md`) | `skills/engineering/setup-matt-pocock-skills/SKILL.md:24,67,74-82,84-100`; `skills/in-progress/serial-implement/IMPLEMENTER-BRIEF.md:10,18,23`; `skills/engineering/retro/SKILL.md:20,41`; `skills/engineering/code-review/SKILL.md:40` |
| `CLAUDE.md` / `AGENTS.md` (root), seconda scrittura | `setup-ts-deep-modules`: aggiunge una riga "context pointer" a `<packages-root>/README.md`. Regola diversa: `CLAUDE.md` se presente, altrimenti `AGENTS.md`, **creando `AGENTS.md`** se nessuno esiste. | — | Sì | `skills/in-progress/setup-ts-deep-modules/SKILL.md:93,95` |
| `docs/agents/issue-tracker.md` | `setup-matt-pocock-skills` (da template `issue-tracker-github.md` / `-gitlab.md` / `-local.md`, oppure prosa libera per "Other" come Linear) | **Nessuna skill lo apre per percorso.** Lo raggiungono via il blocco `## Agent skills`: `to-tickets`, `to-spec`, `code-review`, `implement-spec`, `triage`, `wayfinder` ("should have been provided to you" / "the tracker doc"); `implement` ("fetch it from the issue tracker") | No (raggiunto tramite il puntatore nel blocco) | Scrittura: `skills/engineering/setup-matt-pocock-skills/SKILL.md:49,68,106-114`. Lettura: `skills/engineering/to-tickets/SKILL.md:11,64`; `skills/engineering/to-spec/SKILL.md:9`; `skills/engineering/code-review/SKILL.md:13,33`; `skills/engineering/implement-spec/SKILL.md:9`; `skills/engineering/triage/SKILL.md:11,68`; `skills/engineering/wayfinder/SKILL.md:29`; `skills/engineering/implement/SKILL.md:9` |
| `docs/agents/triage-labels.md` | `setup-matt-pocock-skills`, solo se `triage` è installata e la Section B è stata eseguita | `serial-implement` **per percorso**, se esiste (altrimenti usa il literal `ready-for-agent`); `triage` via blocco ("The mapping should have been provided to you") | No | `skills/engineering/setup-matt-pocock-skills/SKILL.md:51-57,102,111`; `skills/in-progress/serial-implement/SKILL.md:37`; `skills/engineering/triage/SKILL.md:47` |
| `docs/agents/domain.md` | `setup-matt-pocock-skills` (da template `domain.md`) | Nessuna skill lo apre per percorso; raggiunto via il blocco `### Domain docs` | No | `skills/engineering/setup-matt-pocock-skills/SKILL.md:68,99,112`; template `skills/engineering/setup-matt-pocock-skills/domain.md:1-51` |
| `GLOSSARY.md` (root, o uno per contesto) | `domain-modeling` (lazy, al primo termine risolto); `improve-codebase-architecture` (lazy); `grill-with-docs` e `triage` passando per `domain-modeling` | Per nome, "if it exists": `tdd`, `diagnosing-bugs`. Per nome, senza condizione esplicita: `improve-codebase-architecture`, `pr`, `wait-what`, `domain-modeling`, `serial-implement` (con `rg`, solo quando serve), `codebase-design/DESIGN-IT-TWICE.md`. **Senza nome di file** ("the project's domain glossary"): `to-spec`, `to-tickets`, `triage` | No | Scrittura: `skills/engineering/domain-modeling/SKILL.md:40,62`; `skills/engineering/domain-modeling/GLOSSARY-FORMAT.md:34,58`; `skills/engineering/improve-codebase-architecture/SKILL.md:68-69`; `skills/engineering/grill-with-docs/SKILL.md:7`; `skills/engineering/triage/SKILL.md:80`. Lettura: `skills/engineering/tdd/SKILL.md:10`; `skills/engineering/diagnosing-bugs/SKILL.md:10`; `skills/engineering/improve-codebase-architecture/SKILL.md:25,54`; `skills/engineering/pr/SKILL.md:37`; `skills/productivity/wait-what/SKILL.md:7`; `skills/engineering/domain-modeling/SKILL.md:46`; `skills/in-progress/serial-implement/SKILL.md:21`; `skills/in-progress/serial-implement/IMPLEMENTER-BRIEF.md:10,23`; `skills/engineering/codebase-design/DESIGN-IT-TWICE.md:30`; `skills/engineering/to-spec/SKILL.md:17`; `skills/engineering/to-tickets/SKILL.md:25`; `skills/engineering/triage/SKILL.md:74` |
| `GLOSSARY-MAP.md` (root, solo multi-context) | `domain-modeling` (formato); `setup-matt-pocock-skills` lo propone solo con segnali di monorepo | `domain-modeling` (if exists); `wait-what` ("follow `GLOSSARY-MAP.md` ... if the repo has more than one") | No | `skills/engineering/domain-modeling/SKILL.md:24`; `skills/engineering/domain-modeling/GLOSSARY-FORMAT.md:36,56`; `skills/engineering/setup-matt-pocock-skills/SKILL.md:61`; `skills/productivity/wait-what/SKILL.md:7` |
| `docs/adr/NNNN-slug.md` (e `src/<context>/docs/adr/`) | `domain-modeling` (lazy, solo se le tre condizioni sono vere); `improve-codebase-architecture` (offre un ADR quando l'utente rifiuta un candidato con motivo portante); `grill-with-docs` e `triage` via `domain-modeling` | `tdd`, `diagnosing-bugs`, `improve-codebase-architecture`, `to-spec`, `to-tickets`, `triage` ("ADRs in the area you're touching"); `domain-modeling` (scan del numero più alto); `serial-implement` implementer | No | `skills/engineering/domain-modeling/SKILL.md:40,66-74`; `skills/engineering/domain-modeling/ADR-FORMAT.md:3,5,27`; `skills/engineering/improve-codebase-architecture/SKILL.md:14,25,56,70`; `skills/engineering/tdd/SKILL.md:10`; `skills/engineering/diagnosing-bugs/SKILL.md:10`; `skills/engineering/to-spec/SKILL.md:17`; `skills/engineering/to-tickets/SKILL.md:25`; `skills/engineering/triage/SKILL.md:74`; `skills/in-progress/serial-implement/IMPLEMENTER-BRIEF.md:10,23` |
| `CONTEXT.md` / `CONTEXT-MAP.md` | **Nessuna** skill installata (convenzione rinominata upstream, vedi domanda 5) | Nessuna | No | grep: zero occorrenze sotto `skills/` in `HEAD`; presenti invece su `origin/main` del fork |
| `docs/index.md` | Nessuna | Nessuna | No | grep: zero occorrenze in `skills/` |
| `.ai/PROJECT_TRACKING.md` | Nessuna | Nessuna | No | grep: zero occorrenze in `skills/` |
| `.ai/PROJECT_ARCHITECTURE.md`, `.ai/DESIGN_KIT.md` | Nessuna | `serial-implement` (con `rg`, solo quando una decisione della sessione principale ne ha bisogno) | No | `skills/in-progress/serial-implement/SKILL.md:21` |
| `CODING_STANDARDS.md` (fuori dalla lista del ticket) | Nessuna scrive direttamente; `retro` **propone** regole da metterci | `code-review` (obbligatorio se esiste, insieme a `CONTRIBUTING.md`) | No ("read during review, not implementation") | `skills/engineering/code-review/SKILL.md:40`; `skills/engineering/retro/SKILL.md:19,42` |
| `.out-of-scope/*.md` (knowledge base, non istruzioni) | `triage` (richieste enhancement rifiutate) | `triage` (step 1, sempre) | No | `skills/engineering/triage/SKILL.md:74,89`; `skills/engineering/triage/OUT-OF-SCOPE.md:72,84` |
| `GLOSSARY.md` nella cartella corrente di `/teach` (collisione di nome) | `teach` (glossario del workspace di apprendimento) | `teach` | No | `skills/productivity/teach/SKILL.md:12`; `skills/productivity/teach/GLOSSARY-FORMAT.md:1-3` |

Nota: la descrizione (frontmatter) di ogni skill installata è anch'essa nel contesto di ogni sessione (`skills/productivity/writing-for-agents/SKILL.md:24`: "Context load is the cost of always-loaded material ... a skill description"), ma non è un file del progetto.

## Risposte

### 1. Cosa scrivono nei file di istruzioni del progetto

- **`setup-matt-pocock-skills`** è l'unica skill che configura il progetto per le altre. Scrive:
  - il blocco `## Agent skills` con i sottotitoli `### Issue tracker`, `### Triage labels` (solo se `triage` è installata), `### Domain docs`, ciascuno con una riga di sintesi e `See docs/agents/<file>.md` (`skills/engineering/setup-matt-pocock-skills/SKILL.md:84-102`);
  - `docs/agents/issue-tracker.md`, `docs/agents/domain.md`, `docs/agents/triage-labels.md` (SKILL.md:68,106-114);
  - su GitHub/GitLab crea anche le label mancanti con `gh label create` / `glab label create` (SKILL.md:104).
  - Regola di scelta del file confermata testualmente: "If `CLAUDE.md` exists, edit it. Else if `AGENTS.md` exists, edit it. If neither exists, ask the user which one to create; don't pick for them." e "Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa)" (SKILL.md:76-80). Se il blocco esiste già, "update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections." (SKILL.md:82).
  - La skill è `disable-model-invocation: true` (SKILL.md:4): parte solo se l'utente scrive `/setup-matt-pocock-skills`.
- **`setup-ts-deep-modules`** (installata, in `in-progress/`) è il secondo scrittore del file di istruzioni root: aggiunge una riga che punta a `<packages-root>/README.md`. La sua regola diverge: crea `AGENTS.md` se nessuno dei due esiste, invece di chiedere (`skills/in-progress/setup-ts-deep-modules/SKILL.md:93`).
- **`domain-modeling`** scrive `GLOSSARY.md` (o `GLOSSARY-MAP.md` + un `GLOSSARY.md` per contesto) e `docs/adr/NNNN-slug.md`, entrambi in modo lazy (`skills/engineering/domain-modeling/SKILL.md:40`). Ci arrivano anche `grill-with-docs` (che chiama `grilling` + `domain-modeling`, `skills/engineering/grill-with-docs/SKILL.md:7`) e `triage` (SKILL.md:80).
- **`improve-codebase-architecture`** aggiunge termini a `GLOSSARY.md` e offre ADR (SKILL.md:68-70).
- **`retro`** non scrive: presenta candidati (`skills/engineering/retro/SKILL.md:25`), tra cui spostare istruzioni da `AGENTS.md` a `CODING_STANDARDS.md` o a check automatici.
- **Nessuna** skill installata scrive `CONTEXT.md`, `CONTEXT-MAP.md`, `docs/index.md`, `.ai/PROJECT_TRACKING.md` o altro sotto `.ai/`. `docs/index.md` e `.ai/PROJECT_TRACKING.md` su Paddock sono convenzioni del progetto e del `CLAUDE.md` globale dell'utente, portate dentro il flusso Pocock solo dal testo personalizzato di `Paddock/docs/agents/*.md`.

### 2. Cosa leggono, da dove e con quale condizione

Il dato strutturale più importante: **nessuna skill installata apre `docs/agents/issue-tracker.md` o `docs/agents/domain.md` per percorso**. Le skill consumer scrivono "The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`" (`code-review/SKILL.md:13`, `to-tickets/SKILL.md:11`, `to-spec/SKILL.md:9`, `implement-spec/SKILL.md:9`; simile `wayfinder/SKILL.md:29`, `triage/SKILL.md:47`) oppure "the tracker doc" / "the issue-tracker config" (`code-review/SKILL.md:33`, `triage/SKILL.md:11`, `wayfinder/SKILL.md:29`). Il percorso di lettura è quindi:

1. l'harness carica `CLAUDE.md` / `AGENTS.md`;
2. il blocco `## Agent skills` contiene `See docs/agents/issue-tracker.md` (ecc.);
3. l'agente segue quel puntatore quando una skill gli chiede il tracker.

Senza il blocco, i file `docs/agents/*.md` non sono raggiungibili da nessuna skill tranne una: `serial-implement` legge `docs/agents/triage-labels.md` per percorso, "when the repo has one, else the literal `ready-for-agent`" (`skills/in-progress/serial-implement/SKILL.md:37`).

La pagina di documentazione upstream dice invece che le skill "read `docs/agents/issue-tracker.md` at run time" (`docs/engineering/setup-matt-pocock-skills.md:5`); i `SKILL.md` non nominano quel percorso. Il comportamento reale dipende dal puntatore nel blocco.

Glossario e ADR, per condizione:

| Skill | Cosa legge | Condizione | Sorgente |
|---|---|---|---|
| `tdd` | `GLOSSARY.md`, ADR dell'area | "if it exists" | `skills/engineering/tdd/SKILL.md:10` |
| `diagnosing-bugs` | `GLOSSARY.md`, ADR dell'area | "if it exists" | `skills/engineering/diagnosing-bugs/SKILL.md:10` |
| `improve-codebase-architecture` | `GLOSSARY.md`, `docs/adr/` | sempre, "first" | `skills/engineering/improve-codebase-architecture/SKILL.md:14,25` |
| `pr` | `GLOSSARY.md` (vocabolario) | sempre, nessuna condizione | `skills/engineering/pr/SKILL.md:37` |
| `wait-what` | `GLOSSARY.md`, `GLOSSARY-MAP.md` | sempre / se ci sono più contesti | `skills/productivity/wait-what/SKILL.md:7` |
| `domain-modeling` | `GLOSSARY-MAP.md`, `GLOSSARY.md`, `docs/adr/` | if exists, poi crea lazy | `skills/engineering/domain-modeling/GLOSSARY-FORMAT.md:56-58`; `ADR-FORMAT.md:27` |
| `to-spec`, `to-tickets`, `triage` | "the project's domain glossary", ADR | nessun nome di file | `to-spec/SKILL.md:17`; `to-tickets/SKILL.md:25`; `triage/SKILL.md:74` |
| `serial-implement` | `GLOSSARY.md`, `.ai/PROJECT_ARCHITECTURE.md`, `.ai/DESIGN_KIT.md`, ADR (con `rg`); implementer: `CLAUDE.md`, `GLOSSARY.md`, ADR | solo quando una decisione ne ha bisogno / sempre per l'implementer | `skills/in-progress/serial-implement/SKILL.md:21`; `IMPLEMENTER-BRIEF.md:10,18,23` |
| `code-review` | `CODING_STANDARDS.md`, `CONTRIBUTING.md`, ogni doc di standard | obbligatorio se esiste | `skills/engineering/code-review/SKILL.md:40` |
| `setup-matt-pocock-skills` | `AGENTS.md`, `CLAUDE.md`, `GLOSSARY.md`, `GLOSSARY-MAP.md`, `docs/adr/`, `docs/agents/`, `.scratch/` | sempre (esplorazione) | `skills/engineering/setup-matt-pocock-skills/SKILL.md:21-30` |

Il template `domain.md` istruisce anche le skill a leggere `GLOSSARY.md`/`GLOSSARY-MAP.md`/`docs/adr/` "Before exploring" e, se mancano, a "proceed silently" (`skills/engineering/setup-matt-pocock-skills/domain.md:5-11`).

### 3. Cosa presuppongono del root `CLAUDE.md` / `AGENTS.md`

- Che contenga il blocco `## Agent skills` con questi sottotitoli esatti: `### Issue tracker`, `### Triage labels` (opzionale), `### Domain docs`, ognuno con una riga di sintesi e un puntatore `See docs/agents/<file>.md` (`skills/engineering/setup-matt-pocock-skills/SKILL.md:86-100`). È questo il **contratto del blocco**: è l'unico canale con cui le skill consumer trovano tracker, label e layout dei doc di dominio.
- Le skill consumer non controllano i sottotitoli: presuppongono che il tracker sia "already provided" e, se non lo è, mandano l'utente a eseguire `/setup-matt-pocock-skills`.
- `setup-matt-pocock-skills` stessa cerca "Is there already an `## Agent skills` section in either?" (SKILL.md:24) e aggiorna in place: una nuova skill che scrive in `CLAUDE.md` deve quindi lasciare intatto quel blocco, perché una nuova esecuzione del setup lo riscrive per titolo.
- La pagina docs upstream conferma il criterio di successo: "An `## Agent skills` section appears in the instruction file your harness reads, with a one-line summary pointing at each of those files" (`docs/engineering/setup-matt-pocock-skills.md:87`) e "Nothing in the skill files themselves changed" (`:90`).
- Lacuna nota upstream: la scelta del file dipende da quale file esiste, non da quale harness è in uso; con un `CLAUDE.md` residuo, Codex non legge il blocco (`docs/engineering/setup-matt-pocock-skills.md:63`). Paddock aggira il problema con `AGENTS.md -> CLAUDE.md` (symlink).
- `retro` presuppone che `CLAUDE.md`/`AGENTS.md` siano "pushed to the context window of any agent working in this repo" e vadano usati "incredibly sparingly, usually only for navigation pointers" (`skills/engineering/retro/SKILL.md:41`).

Esempio reale, Paddock (`/Users/luca/Projects/Paddock/CLAUDE.md:164-176`): il blocco rispetta la forma (tre sottotitoli, una riga, `See docs/agents/...`), ma le righe di sintesi sono personalizzate: Linear con ID in `.ai/PROJECT_TRACKING.md`, label group `triage`, e `CONTEXT.md` + `docs/adr/` "registered in `docs/index.md`". Il blocco e i file `docs/agents/` sono stati scritti il 2026-09-09 (commit Paddock `a327a60` "docs: configure mattpocock/skills for Linear and domain docs") e poi personalizzati a mano: `docs/agents/issue-tracker.md` è prosa "Other" per Linear; `docs/agents/domain.md` sostituisce `GLOSSARY.md` con `CONTEXT.md` e aggiunge una sezione "This repo".

### 4. Istruzioni in ogni sessione vs documenti su richiesta

- **Ogni sessione**: solo `CLAUDE.md` / `AGENTS.md` della root (li carica l'harness), quindi anche il blocco `## Agent skills` e qualunque riga di puntatore aggiunta da `setup-ts-deep-modules`. Inoltre le descrizioni delle skill installate (non sono file del progetto).
- **Su richiesta** (raggiunti tramite puntatore o perché una skill li nomina): `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`, `docs/agents/domain.md`, `GLOSSARY.md`, `GLOSSARY-MAP.md`, `docs/adr/*`, `CODING_STANDARDS.md`, `.out-of-scope/*`, `.ai/PROJECT_ARCHITECTURE.md`, `.ai/DESIGN_KIT.md`.
- `docs/index.md` e `.ai/PROJECT_TRACKING.md`: nessuna skill Pocock li legge; su Paddock vengono raggiunti solo perché il testo di `CLAUDE.md` e di `docs/agents/*.md` (scritto o modificato a mano) li nomina.

Conseguenza pratica: il contenuto di `docs/agents/*.md` costa contesto solo quando una skill lo richiede; ogni riga nel blocco `## Agent skills` invece costa contesto in ogni sessione.

### 5. Divergenze tra copie personalizzate e upstream che toccano questi file

**`GLOSSARY.md` vs `CONTEXT.md`: non è una personalizzazione del fork, è una rinomina upstream.**

- Upstream ha rinominato la convenzione `CONTEXT.md`/`CONTEXT-MAP.md` in `GLOSSARY.md`/`GLOSSARY-MAP.md` nel commit `d80fa0f` (2026-08-15), mergiato in `upstream/main` con `e484a80` (2026-09-24, "Merge #876: rename CONTEXT.md convention to GLOSSARY.md"), rilasciato in v1.3. Il messaggio di commit elenca le skill toccate: `domain-modeling`, `grill-with-docs`, `improve-codebase-architecture`, `setup-matt-pocock-skills`, `triage`, `tdd`, `diagnosing-bugs`, `ask-matt`, `codebase-design`, `wait-what`; `CONTEXT-FORMAT.md` diventa `GLOSSARY-FORMAT.md`.
- Istruzione di migrazione upstream (`CHANGELOG.md:46` nel fork a `HEAD`): "If you have an existing `CONTEXT.md` (or `CONTEXT-MAP.md`) from before this change, `git mv` it to the new name: the skills only look for `GLOSSARY.md`/`GLOSSARY-MAP.md` going forward."
- Il fork ha mergiato upstream v1.3.1 il 2026-10-07 (`8fbc8f3`) e poi ha allineato la sua skill `serial-implement` (`58dc3a1`, "refactor: point serial-implement at GLOSSARY.md").
- **Dipende dal branch**: su `origin/main` del fork (`cce0aeb`) le skill usano ancora `CONTEXT.md` e `CONTEXT-FORMAT.md` (`git grep` conta occorrenze in 15 file sotto `skills/`). Sul branch installato (`feat/italian-conversation-language` @ `99405cb`) le occorrenze sotto `skills/` sono zero. Quale nome è "installato" dipende quindi da quale branch è checked out in `/Users/luca/Projects/my-pocock-skills`.
- **Paddock è fuori sincronia**: è stato configurato il 2026-09-09 con la convenzione `CONTEXT.md` (allora valida sul fork). Oggi le skill installate scrivono e cercano `GLOSSARY.md`. Possibile guasto, "doppio glossario": `domain-modeling` non trova `GLOSSARY.md` e ne crea uno nuovo accanto a `CONTEXT.md` (`skills/engineering/domain-modeling/GLOSSARY-FORMAT.md:58`); `tdd`/`diagnosing-bugs` con "if it exists" potrebbero saltarlo. Se l'agente corregge seguendo il blocco di Paddock (che dice `CONTEXT.md`) non è verificabile staticamente.

**Personalizzazioni del fork rispetto a `upstream/main`** (`git diff upstream/main HEAD -- skills`, 13 file):

- `serial-implement` (skill solo del fork, 3 file nuovi): l'unica differenza che tocca i file di questa ricerca. Legge `CLAUDE.md`, `GLOSSARY.md`, ADR, `.ai/PROJECT_ARCHITECTURE.md`, `.ai/DESIGN_KIT.md`, e `docs/agents/triage-labels.md` per percorso (`skills/in-progress/serial-implement/SKILL.md:21,37`; `IMPLEMENTER-BRIEF.md:10,18,23`). È anche l'unica skill che nomina un file sotto `.ai/`.
- Regole di lingua italiana in `ask-matt`, `code-review`, `implement`, `tdd`, `to-spec`, `to-tickets`, `triage` (+ `AGENT-BRIEF.md`), `wayfinder`: sezioni `## Conversation language` / `## Ticket language` e il disclaimer di triage tradotto. Non toccano nessun file di istruzioni del progetto, e lasciano invariate label e campi del tracker.
- Nessuna differenza in `setup-matt-pocock-skills`, nei suoi template, in `domain-modeling` o nel contratto del blocco `## Agent skills`.

## Non determinabile

- **Comportamento a runtime del puntatore**: se e quanto affidabilmente un agente segue `See docs/agents/issue-tracker.md` dal blocco quando una skill dice "should have been provided to you". Dai testi si ricava solo che è l'unico canale.
- **Riconciliazione `CONTEXT.md`/`GLOSSARY.md` su Paddock**: se una skill che cerca `GLOSSARY.md` per nome (`tdd`, `diagnosing-bugs`, `pr`, `wait-what`, `improve-codebase-architecture`) userebbe `CONTEXT.md` seguendo il blocco di Paddock, o creerebbe/salterebbe il file. Richiede una prova reale.
- **Stato upstream dopo il 2026-10-07**: l'ultimo fetch locale di `upstream` è del 2026-10-07 (`.git/FETCH_HEAD`); non ho fatto fetch, quindi modifiche upstream successive non sono coperte.
- **Quale branch del fork sarà installato in futuro**: oggi è `feat/italian-conversation-language`, non mergiato in `origin/main`; se si torna su `main` prima del merge, le skill tornano alla convenzione `CONTEXT.md`.
- **Comportamento di caricamento dell'harness**: "Claude Code legge `CLAUDE.md`, Codex legge `AGENTS.md`" si appoggia a `docs/engineering/setup-matt-pocock-skills.md:63` e a `skills/engineering/retro/SKILL.md:41`, non a documentazione ufficiale degli harness, che non ho consultato.
