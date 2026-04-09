# Sharkcode OS — Changelog

## v4.1 (2026-04-09) — Firma Digitale + Legal Pipeline (Phases 73-75)

### Nuove funzionalita

| Cosa | Dettaglio |
|------|-----------|
| Firma digitale DocuSeal | Upload PDF, campi firma posizionati sull'ultima pagina, signing sequenziale (Michel primo, cliente secondo) |
| Signature routes (377 LOC) | `send-for-signature`, webhook, polling, download. Lazy SDK init, SSRF prevention |
| 3-step legal flow | HTML checkpoint → PDF generation → PDF checkpoint → DocuSeal invio |
| Coordinate firme per documento | Contratto e DPA hanno coordinate diverse, calcolate dalla geometria dell'ultima pagina |
| getSignatureById | Query DB per download singola firma |
| Cover gradient base64 | cover-gradient.png embeddato come base64 nei documenti compilati |
| Checkpoint dashboard | Question visibile nella card, link PDF (Contratto/DPA/Proposta/Fattura) |
| Language checker HTML | 6 regole per non segnalare falsi positivi su spaziature CSS, tabelle, prezzi barrati |

### Bug risolti (Phase 75 — 12 fix chirurgici)

| Bug | Fix |
|-----|-----|
| DATE_FULL mancante nelle firme | Aggiunto a buildContrattoPlaceholders e buildDpaPlaceholders |
| extractDeliveryTime perso | Funzione ristrippa "dalla ricezione/dall'avvio/dalla firma" |
| buildPriceTableRows hardcoded | Fallback: parse prezzi dalle sezioni brief |
| cover-gradient.png mancante | Ripristinato da git history (1.4MB PNG) |
| getSignatureById mancante da db.ts | Aggiunta funzione, import in signature.ts risolto |
| Checkpoint fields inconsistenti | clientId/question/created_at (non client/description/created) |
| Concern stringa crash | typeof raw === "string" handling |
| test-* client DocuSeal skip | Skip invio firma per client di test |
| NUM_PAGES corruption non-sanitario | Fix regex page count contratto |
| Auto-approve su timeout 24h | Reject invece di approve |
| Content-Disposition header injection | Sanitize filename |
| external_id con dashed client IDs | Separatore :: invece di - |
| Shell metacharacters in invokeAgent | Escape command string |
| XSS in markdown rendering | DOMPurify sanitization |
| Interval leak su component unmount | clearAllIntervals |
| Dismissed conflicts riappaiono | Persist locale |

### Sicurezza

| Fix | Dettaglio |
|-----|-----------|
| SSRF prevention | Validate download URL in signature route |
| DOMPurify | Sanitize markdown HTML rendering |
| Content-Disposition | Sanitize filename header |
| Shell escape | Escape metacharacters in agent spawn |

### Numeri

| Metrica | Valore |
|---------|--------|
| Commit totali | 870+ |
| Fasi completate | 75 |
| signature.ts | 377 LOC (nuovo) |
| compile-legal.ts | 1.166 LOC |
| run-pipeline.ts | 2.167 LOC |
| db.ts | 2.028 LOC |

---

## v3.6.1 (2026-04-06) — Email Enhancement (Phase 65)

### Nuove funzionalita

| Cosa | Dettaglio |
|------|-----------|
| Cartelle email | Sync IMAP multi-folder: Inbox, Sent, Drafts, Trash con tab nella UI |
| Elimina email | Sposta nel cestino via IMAP, delete permanente se gia nel cestino |
| Salva inviate | Dopo invio SMTP, append alla cartella Sent via IMAP + salvataggio locale con firma |
| Allegati | Parsing da bodyStructure, download on-demand, indicatore nella lista, upload su invio (multipart) |
| AI Compose | "Scrivi con AI": genera bozza da punti chiave, tono, lingua. Usa Claude CLI con Max subscription |
| AI Migliora | "Migliora con AI": riscrive testo esistente mantenendo significato, appare dopo 10+ caratteri |
| Regole team AI | Copywriting + Sales + Marketing iniettati nel prompt: tono consultivo, no aziendalese, accenti corretti |
| Firma professionale | v8 minimal: icona 48px, bordo blu sottile 2px, contatti tutti blu, compatibile dark mode |
| Mark as read | PATCH endpoint, persistente (sync non sovrascrive piu lo stato letto) |
| .env loading | Caricamento automatico da directory server, mapping SITEGROUND_EMAIL |
| UTF-8 | textEncoding base64 per accenti italiani, quoted-printable decoding corretto |

### Bug risolti

| Bug | Fix |
|-----|-----|
| Accenti come rombi nelle email inviate | textEncoding: 'base64' in nodemailer |
| Pallino blu ritorna dopo navigazione | MAX(emails.read, ?) nel sync preserva stato letto |
| Invio email "failed to fetch" | Rimosso API key check (CORS basta per localhost) |
| Claude CLI "Credit balance too low" | Rimosso ANTHROPIC_API_KEY dall'env spawn, usa Max subscription |
| Firma non salvata nelle Inviate | appendToSent riceve htmlBody con firma, non body raw |
| Dropdown email sfondo bianco | Sfondo scuro #111827, rimosso bordo |

### File chiave

- `tools/dashboard/apps/server/src/email-client.ts` — multi-folder sync, delete, sent-append, attachments, firma
- `tools/dashboard/apps/server/src/routes/email.ts` — endpoint DELETE, folders, attachments, multipart send
- `tools/dashboard/apps/server/src/routes/ai.ts` — NUOVO: Claude CLI con regole team Sharkcode
- `tools/dashboard/apps/client/src/views/EmailView.vue` — riscrittura completa con tab, delete, allegati, AI

---

## v3.6 (2026-04-05) — CRM-Driven Sales Pipeline

### Nuove funzionalita

| Cosa | Dettaglio |
|------|-----------|
| CRM State Machine | 7 stati (nuovo, contattato, proposta_inviata, in_trattativa, accettato, cliente, perso) con transizioni validate. Rollback per correzione errori. Conferma per azioni irreversibili. Side effects automatici per transizione. |
| Proposal per Lead | Pipeline `--lead <id>` estrae la sezione specifica da lead-research.md, genera brief automatico con dati CRM + Cialdini framework, output in cartella per-lead (`sales/lead-{id}-{slug}/`). Lancio diretto dal CRM. |
| CRM Completo | Storico contatti con tipo/esito/prossima azione. Documenti strutturati collegati a ogni lead. "Cosa fare oggi" con follow-up scaduti, proposte stale, lead inattive. Filtri ricerca. Dettaglio prospect/client. |
| Pipeline Labeling | Automazioni mostrano tipo pipeline (Ricerca/Proposta/Legale/Produzione), nome lead, link diretto al CRM. Badge colorati per tipo. |
| Pipeline History & Documents | Pipeline registra documenti e eventi direttamente nel CRM. Timeline unificata contacts + documents ordinata per data. Archiviazione storico (move, non delete). |
| 4 stat card CRM | Lead, Prospect, Clienti, Valore pipeline (28.100 EUR da 11 lead Roma Nord). |
| Kanban drag-drop | 5 colonne con trascinamento validato dalla state machine. Score consistente tabella/kanban. |
| Dettaglio lead | Info grid, report research espandibile, timeline unificata, form contatto (input sopra + dropdown sotto), note, prossima azione, documenti collegati. |
| FAB nuovo lead | Pulsante "+" in basso a destra con form campi reali (email, telefono, sito, pacchetto). |
| PageHeader rimosso | Tutte le 14 view a filo sidebar. Contenuto premium senza titoli ridondanti. |

### Bug risolti

| Bug | Causa | Fix |
|-----|-------|-----|
| Status bypass via PATCH | `updateLead()` permetteva di settare status direttamente, aggirando la state machine | Rimosso 'status' dai campi consentiti. Creato `updateLeadWithStatus()` per uso interno. |
| Form lead con contact_info generico | Form inviava `contact_info` ma DB ha `contact_email`/`contact_phone` | Campi separati: email, telefono, sito, pacchetto |
| XSS via v-html | `renderMarkdown` iniettava HTML non sanitizzato | Strip HTML tags prima di rendering |
| addNote rotto per client | Chiamava `/api/leads/{id}/note` per client (non esiste) | Usa `/api/clients/{id}/contact` con tipo 'note' per client |
| null < Date.now() sempre true | Confronto followup date con null era sempre true | Null check esplicito prima del confronto |
| Report endpoint path sbagliato | PROJECT_ROOT calcolato 5 livelli su, ma routes/ e 6 livelli dalla root | Aggiunto un `..` |
| Pipeline spawn apre terminale | Windows mostra finestra terminale per processo spawned | `windowsHide: true` in spawn options |
| Storico pulito ritorna | Polling ricarica dati dal disco | Endpoint DELETE che archivia (move) state files in orchestrator/archive/ |
| Contact form layout rotto | Select occupava tutto lo spazio flex | Input su riga sopra, dropdown+button su riga sotto |
| Score inconsistente | Tabella e kanban mostravano score diversi | `Math.ceil(score/2)` uniforme |
| Cascade delete mancante | `deleteLead` non puliva tabelle correlate | Cascade delete su contact_history e crm_documents |
| Fetch transizioni sequenziale | Ogni apertura dettaglio faceva N chiamate sequenziali | `Promise.all` per transizioni parallele |

### Tabelle DB aggiunte

- `contact_history`: type, summary, outcome, next_action, next_followup_date (per lead)
- `crm_documents`: type, title, file_path, sent_date (per lead)
- `leads`: 10 nuove colonne (updated_at, pipeline_id, documents, value, package, loss_reason, next_action, next_followup_date, proposal_sent_date)

### Dashboard UX

- Colori desaturati ovunque, righe alternate, no linee bianche divisorie
- Nomi lead in testo muted (non grassetto bianco), link ambra
- Sezioni Automazioni: label uppercase 11px muted, non h2 bold
- Pulsanti "Pulisci storico" e "Pulisci risolti" in Automazioni
- Report lead-research con espandi/comprimi nel dettaglio

### File chiave

- `tools/dashboard/apps/server/src/lead-state-machine.ts` (NUOVO)
- `tools/orchestrator/lead-utils.ts` (NUOVO)
- `tools/dashboard/apps/server/src/db.ts` — 3 nuove tabelle, 10 nuove colonne, 8 nuove funzioni
- `tools/dashboard/apps/server/src/routes/business.ts` — 20+ nuovi endpoint CRM
- `tools/dashboard/apps/client/src/views/CrmView.vue` — CRM completo
- `tools/dashboard/apps/client/src/views/AutomazioniView.vue` — pipeline labeling
- `tools/orchestrator/run-pipeline.ts` — --lead flag, brief auto, CRM sync

---

## v3.5 (2026-04-05) — Full Pipeline E2E & Sales Intelligence

### Nuove funzionalità

| Cosa | Dettaglio |
|------|-----------|
| Fasi lead + proposal | Pipeline estesa con fasi pre-vendita: ricerca prospect (lead) e creazione proposta/contratto (proposal). Skippate automaticamente per clienti esistenti (`skipForExisting`). |
| Source audit bloccante | Se il source audit rileva output FAKE (dati inventati), la pipeline ora blocca e ritenta automaticamente invece di proseguire. |
| Strumenti team potenziati | Tavily (ricerca web avanzata) e Playwright (analisi visiva siti) aggiunti a Sales, Lead-Research, Design, SEO, Marketing, Legal, Security, QA, Accessibility. Context7 aggiunto a Development. |
| Sales prompt rinforzato | Obbligo ricerca web reale (Tavily search + extract + crawl), MAI inventare dati, concerns obbligatorie nel frontmatter. Workflow ricerca lead documentato. |
| Pipeline state immediato | Lo stato `running` viene scritto subito all'avvio della pipeline, non dopo il primo step. La dashboard mostra la pipeline dal momento in cui parte. |
| Automazioni: modal output | Pulsante "Vedi output" apre modal full-screen scrollabile con output completo. Badge source audit (Verificato/Dati falsi/Superficiale) dal server. Approva/Rifiuta nel footer del modal. |
| Automazioni: fasi complete | Server aggiornato: pipeline-status ora include tutte le 9 fasi (lead, proposal, research, strategy, design, legal, build, review, audit). |
| Pricing dinamico | Sales non copia prezzi vecchi (Valerio/Mary). Deve ricercare mercato e calcolare pricing per settore/zona. Formato preventivi invariato. |
| Interview skip per lead | La fase lead non richiede interview (è ricerca automatica). |
| Changelog dashboard | MILESTONES.md aggiornato con tutte le fasi 37-56 e milestone v3.1-v3.5. |

### Bug critici risolti

| Bug | Causa | Fix |
|-----|-------|-----|
| Agenti non usavano web search/Tavily/Playwright | `APPDATA` e `LOCALAPPDATA` mancanti nell'env allowlist dei subprocess. Claude CLI non trovava il token OAuth Max e falliva silenziosamente sull'autenticazione, usando solo tool locali. | Aggiunte `APPDATA`, `LOCALAPPDATA`, `XDG_CONFIG_HOME`, `XDG_DATA_HOME` a `ENV_ALLOWLIST` in pipeline-utils.ts |
| Dashboard events 400 | Il payload degli eventi veniva inviato come stringa, ma Zod PipelineEventSchema richiede un oggetto `z.record()`. | Payload ora costruito come oggetto in `sendDashboardEvent()` |
| Pipeline non visibile in Automazioni | 1) Lo stato `pipeline-state-{phase}.json` non veniva scritto all'avvio. 2) Le fasi lead/proposal/audit non erano nella lista del server. | State scritto subito dopo inizializzazione. Server aggiornato con tutte le 9 fasi. |
| Source audit falsi positivi | L'auditor dava FAKE a output con ricerca web reale ma senza libri letti. Non distingueva "dati inventati" da "fonti addon non usate". | Prompt auditor aggiornato: ricerca web reale con URL verificabili non è FAKE. Libri sono addon opzionali. |
| Sidebar badge sovrapposto | Avatar team con `-space-x-1` si accumulavano sopra il testo "X team attivi". | Avatar separati in container con `gap-1`, max 4 visibili, `shrink-0` sul testo. |
| Sales timeout 5 min | Con ricerca web reale (Tavily + WebFetch) 5 minuti non bastano. | Timeout Sales e Lead-Research aumentato a 10 minuti (600s). |
| Sales inventava dati | Nessuna istruzione esplicita di fare ricerca web reale nel prompt iniettato. | `invoke-agent.ts`: ricerca web resa OBBLIGATORIA con minimo 3 ricerche, pena rifiuto quality gate. |
| Sales copiava prezzi vecchi | Esempio preventivo con prezzi hardcoded (2.200 EUR) nel prompt. | Prezzi rimossi dall'esempio. Aggiunta sezione "Pricing Strategy" con obbligo ricerca mercato. |
| Source audit falsi SHALLOW | Auditor dava SHALLOW a output con ricerca web reale perché non leggeva libri addon. Badge dashboard fuorviante. | Prompt auditor riscritto: PASS = dati verificabili (web search reali), non dipende da libri. Libri sono addon opzionali. |
| 13 agenti senza istruzioni MCP | Tutti i team tranne Sales avevano accesso a Tavily/Playwright/WebSearch ma zero istruzioni su come usarli. | Aggiunta sezione "Strumenti di ricerca" con workflow concreti a tutti i 13 agenti. Rimosso WebSearch da Development e QA (non ne hanno bisogno). |
| Pipeline state resta "checkpoint" | Dopo approvazione checkpoint, pipeline-state-{phase}.json restava su status "checkpoint" invece di tornare a "running". Dashboard mostrava pulsanti approva/rifiuta obsoleti. | Aggiunto `pipelineState.status = "running"; saveState()` dopo `waitForCheckpoint()`. |
| Regole globali in invoke-agent.ts | Agenti non rispettavano accenti italiani, regole IVA, em dash. Ogni agente doveva avere le regole nel suo prompt. | Regole globali Sharkcode iniettate automaticamente in TUTTI i prompt: accenti, IVA estone/forfettari, encoding, OÜ. |
| Lead-Research/Account-Manager invisibili | Colore non assegnato in dashboard, apparivano senza colore team. | Stesso colore rosso di Sales (sono dello stesso reparto vendite). |
| Link nel modal aprono nella stessa scheda | Cliccando un URL nel modal output, si usciva dalla dashboard. | `marked` configurato con `target="_blank" rel="noopener noreferrer"` su tutti i link. |
| Quality gate cross-read falso positivo | Penalizzava per cross-read di team senza output. | Verifica se cartella team esiste prima di richiedere cross-read. |
| Pipeline state resta checkpoint dopo approvazione | `pipelineState.status` non tornava a "running". | `saveState("running")` dopo `waitForCheckpoint()`. |
| Dashboard Automazioni non si aggiornava | Pipeline attive caricate solo al mount. | Polling `fetchActivePipelines` ogni 10s. |
| detectConflicts crash | Concerns con topic undefined. | Null check `a.concern?.topic`. |
| Output lead duplicato nel modal | Sales e Lead-Research file separati mostrati insieme. | Stesso outputPath, Lead-Research integra nel file Sales. |
| Modal mostra file di altre fasi | Endpoint restituiva tutti i .md senza filtro. | Filtro per fase nel frontmatter del file. |
| Checkpoint di fine fase mancante | Nessun checkpoint dopo completamento fase. | Checkpoint end-of-phase prima di marcare "completed". |
| No checkpoint inter-round per lead | Sales→Lead-Research si fermava inutilmente. | Skip checkpoint inter-round per fase lead. |
| Progress bar pipeline | Non visibile dove si è nel flusso. | Barra 9 fasi con colori per stato (completed/running/checkpoint/pending). |
| Tavily API key mancante | Placeholder `YOUR_TAVILY_API_KEY`. | Configurata remote MCP con key reale + TAVILY_API_KEY in env allowlist. |
| Sharkcode Interno path sbagliato | Brand assets non visibili in dashboard. Path `../` invece di `../../`. | Fix path relativo in index.ts. |
| BrandView firma al posto foto | MichelD.png è la firma calligrafica, non la foto. | Usato avatar-michel.webp. |
| Sales chiedeva permesso | "Posso mandarle un'analisi?" perde il contatto. | Regola: MAI chiedere, MANDA il valore direttamente. |
| Sales "piccola agenzia" | Suona diminutivo, perde autorevolezza. | Solo "studio di design digitale". |
| Sales metteva OÜ nelle email | Crea diffidenza nel target italiano. | Solo "Sharkcode" nelle comunicazioni. OÜ solo nei documenti legali. |
| Sales "Sharkcode Roma" | Se cercano non trovano nulla, pensano scam. | Solo "Sharkcode", fiducia locale da foto+numero+zone specifiche. |
| Prezzi "incluso" nei preventivi | Il cliente non vede il valore delle singole voci. | Costi reali per voce, risparmio calcolato nel pacchetto. |
| IVA/VIES informazioni imprecise | Reverse charge e VIES spiegati male. | Ricerca approfondita: forfettari reverse charge obbligatorio, IVA non detraibile. |
| Email template non responsive | Outlook/Gmail/mobile incompatibili. | VML fallback, media query mobile, role=presentation, preview text, MSO namespace. |
| Email tono plurale/singolare | "Vostro" nel testo, "Rispondi" nel bottone. | Coerenza in Lei formale per professionisti. |
| Sales "mi sono permesso" | Tono troppo diretto o troppo servile. | "Mi sono permesso di analizzare": umiltà + professionalità. |
| Team duplicato nella Map | Stesso team in round diversi: Map sovrascriveva il primo outputPath. | `getTeamConfigForRound()` traccia occorrenze e restituisce la config giusta per round. |
| Proposal troppo monolitica | Un solo round faceva strategia + email + preventivo + contratto. Agente non riusciva. | 5 round sequenziali con checkpoint: Sales strategia → Copywriting testi → Account-Manager email HTML → Sales preventivo HTML → Legal contratto. |
| Logo/foto URL locali nell'email | Agente copiava path file:/// dal template. | URL CDN hosted su sharkcodestudio.com (logo bianco + foto Michel). |
| Sharkcode Interno non visibile in Brand | Path relativo sbagliato nel server. | Fix `../../` in index.ts. Ora tutti i documenti vendita visibili. |
| HTML non visibili nel modal | Endpoint filtrava solo .md. | Aggiunto .html al filtro + endpoint preview-html per anteprima in nuova scheda. |

### Deliverable

| Cosa | Dettaglio |
|------|-----------|
| Template email HTML | `templates/email-primo-contatto.html`: responsive, Gmail/Outlook/Apple Mail compatibile, VML, Lei formale, logo+foto, box problema, CTA soft |
| Progress bar pipeline | Barra 9 fasi con colori per stato in Automazioni |
| Modal output markdown | Rendering HTML con marked, link target blank, filtro per fase |
| Documenti vendita in Brand | GDPR, VIES, Accessibilità, Crescita, Contratto visibili nella dashboard |

---

## v3.4 (2026-04-05) — Quality & Hardening

| Fase | Cosa |
|------|------|
| Phase 55 | Deep Project Audit: 27 finding (1 critical, 5 high, 13 medium, 8 low). Report completo con fix prioritizzati |
| Phase 55.1 | Audit Critical Fixes: teamRequiredSources per 4 team mancanti (crash fix), ConfigSchema estesa con teamTimeouts/interviewQuestions, single config load (da 3 read a 1), cross-boundary imports centralizzati (barrel file), email endpoint auth (x-api-key) |

---

## v3.3 (2026-04-05) — Pipeline Intelligence & Technical Insurance

| Fase | Cosa |
|------|------|
| Phase 53 | Pre-Pipeline Client Interview: interview.ts con domande fase-specifiche in config, conversazione naturale prima degli agenti, risposte iniettate in ogni prompt come `## Michel's Interview Answers`, clarification detection HITL con pause-and-resume |
| Phase 54 | Test Suite & Server Modularization: index.ts da 2883 a 168 LOC, 11 route modules sotto src/routes/, ws-state.ts + middleware.ts, 24 test bun:test per 4 path critici (pipeline, webhook HMAC, OAuth, system health), API.md con 101 endpoint documentati |

---

## v3.2 (2026-04-04) — Integrations & Automation

| Fase | Cosa |
|------|------|
| Phase 48 | n8n Webhook Integration: webhook-db.ts + webhook-dispatch.ts, HMAC-SHA256 validation, 8 route API, WebhookView 3 tab (config, eventi, consegne), retry con backoff esponenziale |
| Phase 49 | Calendar Create Events + Google Meet: createEvent con conferenceDataVersion:1, form inline in CalendarWidget, icona Meet verde, alert con nome contatto + link Meet |

---

## v3.1 (2026-04-04) — OS Refinement & Dashboard Evolution

| Fase | Cosa |
|------|------|
| Phase 37 | Knowledge Base Expansion: 12 nuovi playbook, 31 libri totali, mapping team→libri in config |
| Phase 38 | Event System & DB Hardening: retention 30gg, busy_timeout SQLite, rotazione JSONL, thread archival |
| Phase 39 | MCP & Agent Robustness: URL configurabile, retry con backoff, model override per team, tool allowlist enforcement |
| Phase 40 | Pipeline Resilience: timeout per team (5/10/15min), kill process tree Windows, escalation |
| Phase 41 | Google Calendar Backend: OAuth, token AES-256-GCM in SQLite, googleapis su Bun |
| Phase 42 | Dashboard Data Views: Finanza/Progetti editabili inline, mutazioni server-confirmed |
| Phase 43 | Calendar Frontend & Alerts: widget calendario mensile, Web Push 30min prima, sync WebSocket |
| Phase 44 | Command Intelligence: command palette italiano, activity ticker, dry-run, knowledge search |
| Phase 45 | Agent Audit: 16 prompt riscritti con personalita', metriche, anti-pattern, libri corretti |
| Phase 46 | Email Integration: IMAP/SMTP SiteGround, auto-match CRM, EmailView |
| Phase 47 | Cloudflare Deploy Status: API Cloudflare, DeployView, redeploy da UI |

---

## v3.0 (2026-04-03) — Dashboard Living System

| Fase | Cosa |
|------|------|
| Phase 24-36 | Design token system, glassmorphism, 10+ primitive UI, shark buddy 3D, command palette, 15 dashboard views, dark premium theme, micro-interactions |

---

## v2.3 (2026-04-03) — Production Readiness & Quality Assurance

| Fase | Cosa |
|------|------|
| Phase 16 | E2E dry-run con test-clinic: pipeline struttura validata |
| Phase 17 | 50 test unitari per pipeline core (validate, discussion, handoff, event-log, pipeline-utils) |
| Phase 18-23 | Dashboard audit, type safety, agent quality, monitoring, shared modules, pipeline intelligence |

---

## v2.2 (2026-04-03) — Pipeline Intelligence & Agent Effectiveness

| Cosa | Dettaglio |
|------|-----------|
| Tool allowlist per team | 16 team con tool specifici (WebSearch per marketing, Bash per dev, Read-only per audit) |
| Config-driven allowlists | Definite in config.json, zero code changes per modificare |
| Pipeline override per cliente | `pipeline-override.json` in context/{clientId}/ sovrascrive fasi, team, modelli |
| Event isolation | Ogni evento ha clientId, WebSocket filtra per cliente, pipeline concorrenti isolate |
| Event store su disco | JSONL fallback quando dashboard offline, replay automatico al riavvio |
| WebSocket Origin check | Solo localhost accettato (gap v2.1 chiuso) |

---

## v2.1 (2026-04-03) — Security & Code Quality Hardening

### Sicurezza

| Cosa | Dettaglio |
|------|-----------|
| SQL injection eliminata | Prepared statements in db.ts, sortOrder whitelist. 7 test |
| Path traversal eliminato | SAFE_NAME_REGEX + validatePathInBounds in MCP server. 11 test |
| Permission-mode hardened | Da `--permission-mode auto` a `acceptEdits` + `--allowedTools` per team |
| Env vars allowlist | Solo PATH, HOME, TERM, SHELL passati ai child. Mai intero process.env |
| CORS localhost-only | Dashboard accetta solo richieste da localhost, non wildcard `*` |
| Rate limiting | Token bucket 30 req/min su endpoint critici POST |

### Robustezza

| Cosa | Dettaglio |
|------|-----------|
| Signal handlers | `gracefulShutdown()` su SIGTERM/SIGINT, kill processi child, pulizia temp |
| Race condition fix | `appendFileSync` atomico per discussion thread (era read-then-write) |
| Temp file cleanup | `unlinkSync` in finally + cleanup all'avvio dei file orfani |
| WebSocket backoff | Exponential 3s → 6s → 12s → max 60s, reset su connessione |

### Qualita' codice

| Cosa | Dettaglio |
|------|-----------|
| parseFrontmatter unificato | Da 4 implementazioni a 1 modulo condiviso `utils/frontmatter.ts` |
| PHASE_ORDER eliminato | Usa `VALID_PHASES` da schemas.ts come unica fonte |
| Config esternalizzata | `MODEL_PRICING` e `TEAM_REQUIRED_SOURCES` in config.json |
| Handoff automatico | `generateHandoff()` chiamato dopo ogni fase completata |
| Dead code rimosso | 75 file generici eliminati da dashboard/.claude/ |
| README corretto | Documenta OAuth Max plan, non piu' "API key required" |
| Fase audit | Nuova fase `audit` nella pipeline (security, qa, dev, a11y) |

---

## v2.0 (2026-04-02) — Agent Intelligence + Observability

| Cosa | Dettaglio |
|------|-----------|
| Discussion thread | Agenti leggono i messaggi degli altri team nella fase corrente |
| Handoff injection | Contesto della fase precedente iniettato nel prompt |
| Cross-reading flessibile | Funziona con filename diversi (output vs review) |
| Circuit breaker | 3 tentativi: sonnet → sonnet semplificato → escalation |
| Quality gate | Output validato per contenuto (sezioni, parole, concerns, sources) |
| Model routing | Opus per orchestrator/strategy, sonnet per il resto |
| Cost tracking | Token parsing da stderr, SQLite, API con breakdown per fase/team |
| MCP Knowledge | `list_knowledge` e `search_knowledge` per 19 libri |
| Web Push | Notifica quando checkpoint aspetta approvazione |
| Staleness detection | Alert se progetto fermo 24h+ |
| Team registry | Tutti i 16 agenti conoscono gli altri team |

---

## v1.0 (2026-04-01) — MVP

| Cosa | Dettaglio |
|------|-----------|
| Dashboard foundation | Bun server + SQLite + Vue 3, eventi via Claude Code hooks |
| Team visualization | Gerarchia team, messaggi agenti in tempo reale, modello per sessione |
| Agent protocol | Output strutturato in `.planning/context/{client}/`, validazione |
| Pipeline skill | `/run-pipeline` nativa con 6 fasi (research → review) |
| 16 agent files | Brand, Design, SEO, Marketing, Dev, QA, Security, Legal, Copywriting, Analytics, Accessibility, Orchestrator, Research, Sales, Lead-Research, Account-Manager |

---

*Sharkcode OS v3.4 | Ultimo aggiornamento: 2026-04-05*
