# skill `project-architecture` — verbale delle decisioni

Sessione di grilling del 2026-09-10. Questo file è l'input per scrivere la spec della skill
in un contesto nuovo: la sezione «Contesto» porta i fatti raccolti sulla macchina, il resto
sono decisioni già prese e confermate.

---

## Contesto (fatti verificati durante la sessione)

**Cosa deve fare la skill.** Generare `.ai/PROJECT_ARCHITECTURE.md` nella root del progetto contro
cui viene lanciata. Il file è referenziato da CLAUDE.md/AGENTS.md, quindi caricato in **ogni**
sessione: deve stare entro ~2k token e contenere solo conoscenza **non derivabile dai manifest** —
semantica reale dei comandi, assenze deliberate, wiring runtime, trappole.

**Chi possiede oggi quel percorso.** Il plugin `tdd-red-handoff` (scritto dall'utente, installato in
`~/.claude/plugins`, repo git a sé con `.claude-plugin/`, `commands/`, `bin/`). Spedisce
`.ai/templates/PROJECT_ARCHITECTURE.template.md` (19 KB) e i comandi `/init-architecture`,
`/update-kit`, `/verify-kit`. **Il plugin sta per essere mandato in pensione e questa skill è il
primo atto**: il percorso passa legittimamente alla skill.

**Cosa rompe la transizione.** `bin/verify-kit.sh` legge il file vivo e verifica meccanicamente:
i sette nomi di comando (`test`, `test (focused)`, `typecheck`, `lint`, `format`, `format:check`,
`coverage`) presenti alla lettera nella tabella § Toolchain; la riga `**Project floor:**` uguale a
quella in CLAUDE.md; nessun nome di modello fuori dal § Model Roster. Scritto il file magro,
`/verify-kit` va rosso finché il plugin resta installato. Deciso: si avvisa e si procede.

**Progetti sulla macchina** (`~/Projects`): Paddock e KeyFuture hanno il kit attivo (`.ai/kit.json`);
AgentsConfigTemplate è il template dei progetti nuovi; MailRules è Python senza CLAUDE.md;
Skills è il repo delle skill.

**Numeri di partenza.** 2k token ≈ 8.000 caratteri di markdown.
- Paddock `.ai/PROJECT_ARCHITECTURE.md`: 76 KB (≈19k token). Di cui § Non-Obvious Traps 8,1 KB
  (≈2k token, cioè il budget intero), § Runtime Wiring 20,3 KB, § Toolchain 4,6 KB.
- KeyFuture `.ai/PROJECT_ARCHITECTURE.md`: 44 KB (≈11k token).
- Le quattro categorie volute, alla fedeltà attuale, fanno ~33 KB ≈ 8k token: quattro volte il tetto.
  Il budget non è un obiettivo da centrare, è un vincolo che impone una regola di sfratto.

**La forma-bersaglio esiste già.** `Paddock/CLAUDE.md` righe 83-104, § Non-Obvious Traps: 3.092
caratteri (≈770 token), 13 voci, una riga ciascuna con sintomo + divieto + rimando all'anatomia
(`Details in .ai/PROJECT_ARCHITECTURE.md § Non-Obvious Traps (ASK-167)`). Scritto a mano, in
produzione, già a due livelli. È il modello di riferimento per il formato.

**Trappola nel wiring dei file di istruzioni.** Su Paddock esistono sia `CLAUDE.md`/`AGENTS.md` di
root sia `.ai/CLAUDE.md`/`.ai/AGENTS.md`: questi ultimi sono **generati** da Laravel Boost
(`config/boost.php` → `agents.*.guidelines_path`) e riscritti a ogni `composer update`. La skill non
deve mai scriverci dentro.

**Vincolo aggiunto in corsa.** L'utente ha (o avrà) file CLAUDE/AGENTS nelle sottodirectory dei
progetti: quelli portano le informazioni locali. Il file prodotto da questa skill contiene **solo
conoscenza globale**. Questo sostituisce l'idea di un secondo file «profondo»: il secondo livello
sono i CLAUDE locali.

---

## Identità e packaging
- Canonical versioned source: `project-architecture/` in this repository. The user-level
  installation is the `~/.agents/skills/project-architecture` symlink to that source; the existing
  `~/.claude/skills/project-architecture` symlink continues to resolve through `~/.agents/skills/`.
  This COINE-90 decision supersedes COINE-89's unversioned real-directory installation, with the
  user's explicit authorization on 2026-09-10.
- `disable-model-invocation: true` — riscrive un file di progetto e chiede conferme, non deve
  partire da sola a metà di un altro lavoro.
- SKILL.md = procedura; formato del file e checklist per stack = file di reference separati
  (progressive disclosure).
- Promozione a plugin rimandata: la struttura è già pronta al travaso.

## Il file prodotto
- Destinatario: il modello. **Inglese telegrafico**, struttura `<soggetto>: <fatto> — <conseguenza/azione>`,
  nessuna abbreviazione da decodificare, **nessuna legenda** (il file è letto in sessioni dove la skill
  non è caricata: una legenda costerebbe token a ogni sessione). Traducibile per un umano su richiesta.
  Misurato su una trappola vera: prosa 75 token → telegrafico 40 → notazione a campi 30. Scelto il
  telegrafico: prende l'85% del guadagno a rischio zero di ambiguità.
- Budget dichiarato in testa: `max_tokens_k=2`. Misura = `wc -c` / 4; allarme al 90% (7.200 char).
  Aumento **solo** con incrementi di 0,5 (= 2.000 caratteri) e **solo** se, applicato l'ordine di
  sfratto, restano fuori voci di priorità 1-3. La proposta di aumento elenca quali voci entrerebbero.
- Marcatore in prima riga — paternità e memoria fra un lancio e l'altro:
  `<!-- project-architecture v1 | updated <data> | max_tokens_k=2 | not-populated: … | sources-explored: … | dropped: N -->`
  Marcatore assente = file estraneo. Gli scarti si registrano come **conteggio**, non per nome.
- Scheletro e spesa indicativa (il vincolo duro è il totale, non la singola quota):

```
## Shape                 ~200 tok  identità, modello di layering, mappa layer→dir, deviazioni
## Commands              ~250 tok  cosa incatena ogni meta-comando, e cosa NON significa un rosso
## Deliberately absent   ~150 tok  librerie assenti per scelta + l'alternativa in uso
## Runtime wiring        ~400 tok  file generati, plugin che riscrivono, routing disattivato
## Traps                 ~900 tok  una riga per trappola, ancora verificabile inclusa
```

  `Shape` per prima: risponde alla domanda più frequente («dove metto questo file»). `Traps` per
  ultima: è la sezione che cresce. Intestazione **piena** (identità + layering + mappa + deviazioni),
  perché i CLAUDE.md verranno rivisti tutti e i fatti di progetto migrano qui.

## Regole di contenuto
- **Regola d'ammissione**: entra solo ciò che un agente sbaglia *facendo la cosa di default*, senza
  avere motivo di cercare. `composer test` meta-comando entra (lo lanci comunque); l'ordine dei
  middleware no (ci arrivi solo scrivendo una guardia, e lì hai motivo di cercare).
- **Ordine di sfratto**, dal basso: 5 wiring / 4 trappole a fallimento rumoroso / 3 trappole a
  fallimento silenzioso / 2 omissioni deliberate / 1 semantica dei comandi.
- **Non riempire**: se a T0 una sezione non ha materiale, si omette e si registra nel marcatore.
  Le trappole deboli si aggiungono a Tn, non si inventano a T0.
- **Omissioni deliberate**: solo con evidenza (doc/ADR) o dette dall'utente; mai dedotte dall'assenza
  in `package.json`. Sempre con motivo + alternativa in uso — «zod: assente per scelta, la validazione
  al boundary sta nei Form Requests; non aggiungerlo», mai «zod NOT INSTALLED».
- **Granularità atomica**: una voce = una riga con la sua fonte, perché il merge a Tn sia meccanico.
- **Test globale/locale**: raggio d'azione. Globale se un agente ci inciampa partendo da qualunque
  punto del repo o se l'effetto attraversa più directory; locale se colpisce solo chi lavora già lì.
  I rilievi locali si **elencano soltanto**; la skill chiede se e dove persisterli (file in root, nota
  Notion, altra destinazione raggiungibile dal modello). Non li scrive e non li propone lei.

## Ciclo di esecuzione
1. **Rileva il file esistente.** Marcatore presente → update conservativo. Assente → avvisa che la
   forma verrà sovrascritta, archivia l'originale in `.ai/archive/PROJECT_ARCHITECTURE.<data>.md`,
   e **ne mina il contenuto**: le informazioni non recuperabili altrove vanno preservate.
2. **Scavo misto.** In sessione: il file allo stesso percorso (fonte più ricca, va capita) e i
   CLAUDE/AGENTS. Delegati a subagent (contesto principale pulito): `.ai/plans/` e post-mortem,
   ADR e `docs/`, `git log` (revert, fix ripetuti sullo stesso punto). Ordine fonti: file esistente →
   CLAUDE/AGENTS root e sottodirectory → ADR/docs → plans → git log → manifest e script (solo per
   capire cosa c'è dentro i comandi). Fonte che non produce nulla: registrata nel marcatore come
   esplorata, non riaperta al lancio successivo se non è cambiata.
3. **Scrive il file e chiede validazione.** A schermo elenca solo le decisioni di scarto **in bilico**;
   gli scarti ovvi non si sottopongono.
4. **Duplicazione con CLAUDE.md.** Regola di confine: CLAUDE.md = processo (come si lavora),
   PROJECT_ARCHITECTURE.md = fatti (com'è fatto). La skill copia i fatti nel proprio file e **propone**
   la rimozione da CLAUDE.md — solo delle voci effettivamente scritte, mai di quelle scartate per
   budget. Esegue su conferma.
5. **Gancio.** Se manca in CLAUDE/AGENTS di root: «Non ho trovato un gancio a questo file, lo aggiungo?».
   Mai `.ai/CLAUDE.md`/`.ai/AGENTS.md` (rigenerati da Boost). Riga proposta:
   `Project facts (commands, wiring, traps, deliberate absences): read .ai/PROJECT_ARCHITECTURE.md in full before planning or editing.`
6. **Progetti con kit attivo.** Avvisa che `/verify-kit` fallirà finché il plugin resta installato.
   Scrive comunque: rifiutarsi renderebbe la skill inutile proprio dove c'è più contesto accumulato.

## Update a Tn
- Ogni voce porta un'**ancora verificabile** già nel proprio testo (path, chiave di config, nome di
  comando: `DB_HOST=paddock-mysql`, `resources/js/actions/`, `config/boost.php`). Costo aggiuntivo
  zero: l'ancora è il testo della voce, non un campo in più.
- A Tn la skill fa grep delle ancore: sparita o cambiata → voce **sospetta**, proposta per rimozione o
  riscrittura. **Mai rimozione automatica.**
- Voci non ancorabili (fatti storici, i «perché» di una scelta) = non verificabili, sopravvivono
  finché non le tocca l'utente.

## Assunzioni dichiarate
1. Il file va **committato**, non ignorato: è conoscenza condivisa, e senza storia git l'archivio è
   l'unica rete.
2. La lingua del file è l'**inglese** (discende dal formato scelto).
3. Se `.ai/` non esiste, la skill la crea.
4. Se il progetto non è un repo git, la fonte `git log` si salta e il marcatore la registra come
   non disponibile.

## Collaudo
KeyFuture (44 KB, stesso stack di Paddock, metà volume: prova il ciclo completo incluso il mining di
un file estraneo) → Paddock (76 KB, ~30 candidati, sezione CLAUDE.md da potare: caso peggiore) →
MailRules (nessun CLAUDE.md, nessun kit, un `DECISIONI.md` da 13 KB: prova che la skill non riempia
per riempire).

## Fuori perimetro
Non sono stati creati CONTEXT.md né ADR: il vocabolario della skill (marcatore, ancora, raggio
d'azione, ordine di sfratto) appartiene ai suoi file di reference. Un ADR sensato esiste, ma nei repo
Paddock e KeyFuture: perché `/verify-kit` diventa rosso e perché il kit va in pensione.
