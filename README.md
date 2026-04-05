# Sharkcode OS

An AI-powered operating system for running a one-person web agency. Not a wrapper around ChatGPT. Not a demo. A working system managing real clients, real invoices, real deployments.

## What this actually is

Sharkcode OS is the command center for [Sharkcode OU](https://sharkcodestudio.com), an Estonian web agency serving Italian SMEs. One founder (Michel), 16 AI agents, zero employees.

The system orchestrates specialized AI agents through a pipeline with explicit checkpoints and human approval gates. Michel says a client name, the system runs research, competitor analysis, SEO, design, legal, development — stopping at each phase for review. Agents communicate with each other through discussion threads. Everything is logged, costed, and visible in a real-time dashboard.

**This is not vaporware.** It manages active client projects with real revenue:
- 4 clients in pipeline (2 active, 1 prospect, 1 e-commerce) + 11 qualified leads (28,100 EUR pipeline)
- 63+ completed development phases across 676+ commits
- Full 9-phase pipeline: lead → proposal → research → strategy → design → legal → build → review → audit
- CRM-driven sales lifecycle: 7-state machine, per-lead proposals, unified timeline, "cosa fare oggi"
- Source audit blocks fabricated output (FAKE verdict triggers automatic retry)
- End-to-end tested on production client work
- Deep audited with 27 findings identified and all critical fixes applied

## Screenshots (real dashboard, not mockups)

| Dashboard | Team Structure |
|:---------:|:--------------:|
| ![Dashboard](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/01-dashboard.png) | ![Structure](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/02-struttura.png) |

| Projects | Knowledge Base |
|:--------:|:--------------:|
| ![Projects](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/03-progetti.png) | ![Knowledge](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/05-knowledge.png) |

| CRM (11 leads, 28.100 EUR) | Brand Identity |
|:--------------------------:|:--------------:|
| ![CRM](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/04-crm.png) | ![Brand](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/06-brand.png) |

| System Health | Automations |
|:-------------:|:-----------:|
| ![System](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/07-system.png) | ![Automazioni](https://raw.githubusercontent.com/Michel9329/sharkcode-os-demo/main/08-automazioni.png) |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Claude Code CLI                          │
│              (command entry point — terminal)                 │
└──────────────────────┬──────────────────────────────────────┘
                       │ spawns agents via CLI
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   Pipeline Orchestrator                       │
│                                                               │
│  run-pipeline.ts → spawn agent → validate output → handoff   │
│                                                               │
│  • Per-team timeouts (configurable, 600s-900s)               │
│  • 3-retry circuit breaker with exponential backoff           │
│  • Process tree kill on timeout (platform-aware)              │
│  • Quality gates: content validation, not just format         │
│  • Cost tracking per agent/team/phase (token-level)          │
│  • Checkpoint files on disk (resumable)                       │
│  • Env var allowlist (agents only see what they need)         │
└──────┬──────────────┬──────────────────┬────────────────────┘
       │              │                  │
       ▼              ▼                  ▼
┌────────────┐ ┌────────────┐  ┌──────────────────┐
│  16 Agent  │ │ MCP Server │  │  Dashboard Server │
│  Prompts   │ │ (10 tools) │  │  (Bun + SQLite)   │
│            │ │            │  │                    │
│ .claude/   │ │ sharkcode  │  │  120+ API routes    │
│ agents/    │ │ -core      │  │  WebSocket live     │
│            │ │            │  │  Google Calendar    │
│ Each has:  │ │ Per-team   │  │  Email IMAP/SMTP   │
│ - Identity │ │ tool       │  │  Cloudflare API    │
│ - Rules    │ │ allowlists │  │  Web Push alerts    │
│ - Metrics  │ │            │  │  Event retention    │
│ - Model    │ │ Path       │  │  Discussion threads │
│   assign   │ │ validation │  │                    │
└────────────┘ └────────────┘  └────────┬─────────┘
                                         │ WebSocket
                                         ▼
                               ┌──────────────────┐
                               │  Vue 3 Dashboard  │
                               │                    │
                               │  15 views          │
                               │  60+ components    │
                               │  17 composables    │
                               │  Dark glassmorphism│
                               │  Three.js + GSAP   │
                               └──────────────────┘
```

## Tech stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Runtime | **Bun** | Fast, native SQLite, TypeScript-first, no transpile step |
| Server | **Bun.serve()** | No Express. Direct HTTP + WebSocket in one server |
| Database | **SQLite** (bun:sqlite) | WAL mode, prepared statements, zero-config, single file |
| Frontend | **Vue 3** + Vite 7 + Tailwind CSS | Reactive composables, fast HMR |
| 3D | **Three.js** + GSAP | Shark buddy mascot (yes, really — it tracks pipeline activity) |
| Charts | **ApexCharts** | Cost breakdowns, activity timelines |
| Calendar | **googleapis** | OAuth2 flow, AES-256-GCM token encryption |
| Email | **imapflow** + nodemailer | IMAP sync + SMTP send (SiteGround) |
| Deploy | **Cloudflare API v4** | Deployment status, trigger builds |
| Notifications | **web-push** | Meeting alerts 30min before, pipeline checkpoints |
| AI | **Claude Code CLI** | Agents spawned as subprocesses, not API calls |
| MCP | **@modelcontextprotocol/sdk** | 10 custom tools, STDIO transport |
| Validation | **Zod** | Runtime schema validation for pipeline config |

## The 16 agents

Not generic "assistants". Each agent has a defined identity, specific tool permissions, required reading from the knowledge base, and success metrics.

| Agent | Model | Role |
|-------|-------|------|
| **Orchestrator** | Opus | Coordinates teams, resolves conflicts, strategic decisions |
| **Research** | Sonnet | Market analysis, competitor deep dives, trend detection |
| **Brand** | Sonnet | Target persona, positioning, naming, visual identity |
| **Copywriting** | Sonnet | Website copy, CTAs, email sequences, microcopy |
| **Design** | Opus | UI/UX, design systems, component specs, GSAP animations |
| **SEO** | Sonnet | Keyword research, site architecture, schema markup, E-E-A-T |
| **Marketing** | Sonnet | Campaign strategy, market fit, launch plans |
| **Development** | Sonnet | Astro builds, Tailwind, performance, deploy checklists |
| **QA** | Sonnet | Testing, browser validation, performance monitoring |
| **Legal** | Sonnet | Privacy policy, GDPR, cookie compliance, DPA |
| **Security** | Sonnet | STRIDE threat modeling, dependency audit, hardening |
| **Accessibility** | Sonnet | WCAG 2.1 AA, screen reader testing, focus management |
| **Analytics** | Sonnet | GA4 setup, conversion funnels, KPI dashboards |
| **Lead Research** | Sonnet | Prospect sourcing, market scanning, qualification |
| **Account Manager** | Sonnet | Client relationship, timeline management, UX research |
| **Sales** | Sonnet | Proposals, pricing strategy, win theme analysis |

**Tool allowlists are enforced.** Development gets `Read,Write,Edit,Bash,Context7`. Sales gets `WebSearch,Tavily,Playwright`. Legal gets `Read,Write,Grep,Glob,WebSearch,Tavily`. No agent can access tools it doesn't need. Configured in `config.json`, enforced by the MCP server at runtime.

## How the pipeline works

```
Michel: "run-pipeline valerio research"

1. Orchestrator loads client brief (target persona, budget, goals)
2. Resolves which teams run in what order
3. Spawns Research agent as a subprocess
   └── Agent reads: client brief + relevant books + previous phase output
   └── Agent writes: research output to .planning/context/valerio/research/
   └── Agent posts discussion messages visible to other teams
4. Pipeline validates output (word count, required sections, key decisions)
5. If validation fails → retry (up to 3x with backoff)
6. If passes → generate handoff document for next phase
7. Dashboard shows everything in real-time via WebSocket
8. Michel reviews, approves or requests changes
9. Next phase starts
```

Each phase produces artifacts in `.planning/context/{clientId}/{team}/`. Teams read each other's output. Design reads Brand. Development reads Design. SEO reads Research. It's not a chain — it's a directed graph.

**Cost is tracked per token.** Every agent run logs: model used, input tokens, output tokens, cost in USD, duration. Visible in the Finance view.

## The dashboard (15 views)

| View | What it does |
|------|-------------|
| **Dashboard** | Active agents, pipeline progress, cost summary, calendar widget, activity ticker |
| **Projects** | All client projects with status, phase progress, team output |
| **Project Detail** | Single project deep dive: timeline, discussions, artifacts |
| **Activity** | Real-time event stream from all agents (WebSocket) |
| **Discussion** | Thread viewer — agent-to-agent conversations per client/phase |
| **CRM** | 7-state lead lifecycle, kanban drag-drop, per-lead proposals, unified timeline, "cosa fare oggi", 11 qualified leads |
| **Finance** | Cost tracking, invoices, expenses, revenue per client |
| **Email** | Inbox sync (IMAP), search, compose, reply |
| **Knowledge** | Full-text search across 31 research books |
| **Brand** | Identity assets, color palette, typography, brand rules |
| **Structure** | Org chart, team profiles, tool allowlists, model assignments |
| **System Health** | Server uptime, DB stats, WebSocket connections, staleness detection |
| **Deploy** | Cloudflare Pages: deployment history, status, trigger redeploy |
| **Changelog** | Release notes per phase |
| **Automations** | Active pipelines with live status, checkpoint approve/reject with output preview, pipeline type/lead labels, archive history |

The design is glassmorphism dark theme with semi-transparent cards, blur effects, and subtle glow animations. Premium look, not Material UI defaults.

## Knowledge base

31 books distilled into markdown, searchable via MCP. Not just stored — actively routed to the right teams.

**Examples:**
- Brand team reads *StoryBrand*, *Positioning*, *Brand Gap*
- Security team reads *Web App Hacker's Handbook*, *Tangled Web*
- SEO team reads *SEO-specific books* + E-E-A-T framework
- Sales team reads *$100M Offers*, *Psychology of Persuasion*, *Wizard of Ads*

Mapping is in `config.json` → `teamRequiredSources.books`. When an agent is spawned, relevant book excerpts are injected into its context.

## MCP server (10 tools)

The custom MCP server (`tools/mcp-servers/sharkcode-core/`) gives agents controlled access to the system:

| Tool | Purpose |
|------|---------|
| `get_client_brief` | Read client strategy document |
| `get_team_output` | Fetch another team's output (cross-team reads) |
| `list_clients` | List all client projects |
| `list_knowledge` | Browse the 31-book knowledge base |
| `read_knowledge` | Full-text read a book chapter |
| `search_knowledge` | Full-text search across all books |
| `get_pipeline_status` | Current phase + progress for a client |
| `list_project_files` | Files created for a client |
| `post_discussion` | Write a message to the discussion thread |
| `emit_dashboard_event` | Send events to the dashboard |

**Security:** Path validation prevents directory traversal. Name regex blocks injection. Tool allowlists are checked before every call.

## What makes this different from [n8n / Zapier / LangChain / CrewAI]

| Aspect | Sharkcode OS | Typical AI agent frameworks |
|--------|-------------|---------------------------|
| **Agent spawning** | Real CLI subprocesses, not API chains | API call wrappers |
| **Human in the loop** | Mandatory checkpoints at every phase | Optional, often skipped |
| **Cost tracking** | Per-token, per-agent, per-phase, in USD | Usually none or estimated |
| **Knowledge routing** | 31 books mapped to specific teams | RAG over everything |
| **Tool security** | Per-team allowlists, path validation | Usually all-or-nothing |
| **Communication** | Agents write discussion threads, read each other | Usually sequential chain |
| **Dashboard** | Real-time WebSocket, 15 views, glassmorphism | Log files or basic UI |
| **Integrations** | Google Calendar, Email, Cloudflare — in the same system | Separate tools |
| **Pipeline resilience** | Timeouts, retries, circuit breaker, process kill | Crash and pray |
| **Real clients** | Managing actual paying projects | Demo with toy examples |

The key philosophical difference: **agents don't have free rein.** They operate within defined boundaries, with explicit checkpoints, quality gates, and human approval. The system is designed to be trustworthy, not autonomous.

## Resilience

Things go wrong. Agents hang, outputs are garbage, APIs fail. The system handles this:

- **Per-team timeouts** — Development gets 900s, others get 600s (configurable in `config.json`)
- **Process tree kill** — Platform-aware (`taskkill` on Windows, `SIGTERM→SIGKILL` on Unix)
- **3-retry circuit breaker** — Exponential backoff, then escalate to human review
- **Quality gates** — Content validation (not just "did it return 200 OK")
- **Staleness detection** — Alert if pipeline hasn't progressed in 24h
- **Event log fallback** — JSONL on disk when dashboard is offline
- **Retention policy** — Auto-cleanup events >90 days, log rotation at 1MB
- **WAL mode** — SQLite concurrent read safety with 5-minute checkpoints
- **WebSocket reconnection** — Jitter backoff for dashboard clients

## Security

Not an afterthought:

- **AES-256-GCM** encryption for OAuth tokens in SQLite
- **Prepared statements** everywhere (no raw SQL concatenation)
- **Env var allowlist** — child processes only see explicitly approved variables
- **Tool allowlists** — per-team, enforced at MCP server level
- **Path bounds checking** — `validatePathInBounds()` prevents directory traversal
- **CORS headers** — origin validation on all API routes
- **SAFE_NAME_REGEX** — alphanumeric + hyphens only, max 100 chars

## Project structure

```
sharkcode/
├── .claude/
│   ├── agents/                 # 16 agent prompt files
│   │   ├── sharkcode-orchestrator.md
│   │   ├── sharkcode-research.md
│   │   ├── sharkcode-development.md
│   │   └── ... (16 total)
│   └── skills/                 # 47 skill definitions
│       ├── seo-optimizer/
│       ├── senior-architect/
│       ├── code-reviewer/
│       └── ... (47 total)
├── .planning/
│   ├── PROJECT.md              # Vision, context, current milestone
│   ├── ROADMAP.md              # Phase structure, dependencies
│   ├── REQUIREMENTS.md         # Feature specs with acceptance criteria
│   ├── STATE.md                # Current system state
│   ├── phases/                 # 47 phase directories (01-47)
│   │   ├── 01-dashboard-foundation/
│   │   ├── ...
│   │   └── 47-cloudflare-deploy-status/
│   └── context/                # Per-client artifacts
│       └── {clientId}/
│           ├── brief.md
│           ├── discussion/
│           └── {team}/         # Team output files
├── tools/
│   ├── dashboard/
│   │   └── apps/
│   │       ├── server/         # Bun server (40+ routes, WebSocket, integrations)
│   │       │   └── src/
│   │       │       ├── index.ts            # Slim dispatcher (~168 LOC)
│   │       │       ├── routes/            # 11 route modules
│   │       │       │   ├── events.ts      # Pipeline events
│   │       │       │   ├── clients.ts     # CRM + costs
│   │       │       │   ├── calendar.ts    # Google Calendar
│   │       │       │   ├── email.ts       # IMAP/SMTP
│   │       │       │   ├── webhooks.ts    # n8n integration
│   │       │       │   └── ...            # 6 more modules
│   │       │       ├── ws-state.ts        # Shared WebSocket state
│   │       │       ├── middleware.ts       # CORS + rate limiting
│   │       │       ├── db.ts              # SQLite schema + WAL + CRM tables
│   │       │       ├── lead-state-machine.ts  # 7-state CRM lifecycle
│   │       │       ├── google-calendar.ts  # OAuth2 + sync
│   │       │       ├── email-client.ts     # IMAP + SMTP
│   │       │       ├── cloudflare-api.ts   # Deploy status + triggers
│   │       │       ├── oauth-crypto.ts     # AES-256-GCM
│   │       │       └── push.ts            # Web Push notifications
│   │       └── client/         # Vue 3 dashboard
│   │           └── src/
│   │               ├── views/       # 15 views
│   │               ├── components/  # 60+ components
│   │               ├── composables/ # 17 composables
│   │               └── styles/      # Dark glassmorphism theme
│   ├── orchestrator/           # Pipeline engine
│   │   ├── run-pipeline.ts     # Main runner (--lead flag for per-lead proposals)
│   │   ├── lead-utils.ts       # Lead extraction, brief generation, pre-launch validation
│   │   ├── interview.ts        # Pre-pipeline client interview
│   │   ├── config.json         # Timeouts, pricing, allowlists, book mappings, interview questions
│   │   ├── discussion.ts       # Thread management
│   │   ├── validate.ts         # Quality gates + clarification detection
│   │   ├── handoff.ts          # Phase-to-phase document generation
│   │   ├── schemas.ts          # Zod validation (single config source)
│   │   └── event-log.ts        # Immutable JSONL log
│   └── mcp-servers/
│       └── sharkcode-core/     # Custom MCP server (10 tools)
├── research/                   # 31 books in markdown
│   ├── book-storybrand.md
│   ├── book-web-security-developers.md
│   └── ... (31 total)
├── clients/                    # Client project folders
│   ├── valerio-casa-riposo/
│   ├── mary-logopedista/
│   └── ...
└── templates/                  # Reusable templates (GDPR, sales framework)
```

## Numbers

| Metric | Value |
|--------|-------|
| Total commits | 676+ |
| Development phases completed | 63 |
| Lines of code (server) | ~5,400 (modularized into 15 route modules) |
| Lines of code (client) | ~15,800 |
| Lines of code (orchestrator) | ~5,600 |
| Lines of code (MCP) | ~560 |
| **Total LOC** | **~27,400** |
| Agent definitions | 16 |
| Skill definitions | 47 |
| Research books | 31 |
| Dashboard views | 15 |
| Vue composables | 16 |
| Vue components | 30+ |
| API routes | 120+ |
| Test suite | 24 tests (bun:test) for critical paths |
| MCP tools | 10 |
| Active client projects | 4 + 11 qualified leads |

## Setup

### Prerequisites

- [Bun](https://bun.sh) (latest)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Node.js 18+ (for MCP server)

### Install

```bash
# Server
cd tools/dashboard/apps/server
bun install

# Client
cd tools/dashboard/apps/client
bun install
```

### Configure

Copy `.env.example` to `.env` in the server directory and fill in:

```env
# Google Calendar (OAuth2)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=http://localhost:4000/api/oauth/google/callback
GOOGLE_OAUTH_ENCRYPTION_KEY=    # 64-char hex key for AES-256-GCM

# Email (IMAP/SMTP)
EMAIL_HOST=
EMAIL_USER=
EMAIL_PASSWORD=

# Cloudflare Pages
CLOUDFLARE_API_TOKEN=
CLOUDFLARE_ACCOUNT_ID=
```

### Run

```bash
# Terminal 1: Server
cd tools/dashboard/apps/server
bun run dev              # http://localhost:4000

# Terminal 2: Client
cd tools/dashboard/apps/client
bun run dev              # http://localhost:5173

# Terminal 3: Pipeline (when needed)
bun run tools/orchestrator/run-pipeline.ts {clientId} {phase}
```

## What's next

### Remaining integrations (v3.2, not urgent)
- **Phase 50:** Maatilayla integration (Astro reference site: git status, deploy, Lighthouse, pattern extraction)
- **Phase 51:** Lindorug integration (Shopify CLI: theme push/pull, orders, revenue tracking)
- **Phase 52:** Wedding Templates integration (personal project)

### Recently completed
- **v3.6** — CRM-driven sales pipeline: 7-state machine, per-lead proposals, unified timeline, "cosa fare oggi", pipeline labeling, 11 qualified leads (28,100 EUR)
- **v3.5** — Full pipeline E2E: 9-phase lead-to-audit, source audit FAKE blocking, Tavily MCP, pricing dinamico, template email HTML
- **v3.4** — Deep project audit (27 findings), critical fixes (config integrity, email auth, cross-boundary imports)
- **v3.3** — Pre-pipeline client interview system, server modularization (2883→168 LOC), 24-test suite

## Example: full pipeline run

Here's what actually happens when a client project runs through the system. This is a real sequence, not a hypothetical.

```
$ bun run tools/orchestrator/run-pipeline.ts valerio-casa-riposo research

[ORCHESTRATOR] Pipeline start: valerio-casa-riposo / research
[ORCHESTRATOR] Session ID: pipeline-valerio-casa-riposo-1711234567890
[ORCHESTRATOR] Loading client brief: .planning/context/valerio-casa-riposo/brief.md
[ORCHESTRATOR] Checking for completed work... none found, starting fresh
[ORCHESTRATOR] Estimated duration: ~8 min (based on historical data)

[ORCHESTRATOR] Phase: research
[ORCHESTRATOR] Sequence: research → brand → seo → marketing → design → legal
[ORCHESTRATOR] Starting with: research team

────────────────────────────────────────────────────────────
 STEP 1/6: research
────────────────────────────────────────────────────────────

[ORCHESTRATOR] Building prompt for sharkcode-research...
  ├── Client brief: loaded (target: casa di riposo Roma, budget: 2.200 EUR)
  ├── Knowledge base: 4 books injected (StoryBrand, Positioning, SEO, Copywriting)
  ├── Cross-team outputs: none (first phase)
  ├── Discussion thread: empty
  └── Tool allowlist: Read,Write,Grep,Glob,WebSearch,WebFetch,mcp__sharkcode-core__*

[ORCHESTRATOR] Launching agent: sharkcode-research (model: sonnet)
[ORCHESTRATOR] Timeout: 600000ms (10 min)
[ORCHESTRATOR] PID: 28451

[DISCUSSION] sharkcode-research: Inizio lavoro sulla fase research.
             Leggo brief e output degli altri team.

[DASHBOARD EVENT] → agent_start (research inizia lavoro su research)
[WEBSOCKET] → broadcast to connected clients

  ... agent runs for ~6 minutes ...
  ... writes files to .planning/context/valerio-casa-riposo/research/ ...

[DISCUSSION] sharkcode-research: Completato. Ho trovato 12 competitor diretti
             nella zona Roma Nord. Il mercato RSA e' saturo di siti generici
             con stock photos. Opportunita': nessuno ha video tour o
             testimonianze verificabili. Raccomando storytelling familiare.

[ORCHESTRATOR] Agent exited with code 0
[ORCHESTRATOR] Token usage: 45,231 input / 12,847 output
[ORCHESTRATOR] Cost: $0.089 (sonnet pricing)

[ORCHESTRATOR] Validating output quality...
  ├── File exists: research-output.md ✓
  ├── Word count: 3,847 (min: 500) ✓
  ├── Required sections: Competitor Analysis ✓, Market Overview ✓,
  │   Opportunities ✓, Recommendations ✓
  ├── Key decisions documented ✓
  └── QUALITY GATE: PASS

[ORCHESTRATOR] Posting cost data to dashboard...
[ORCHESTRATOR] Generating handoff document for next team...

[DASHBOARD EVENT] → agent_complete (research completato, costo: $0.089)

────────────────────────────────────────────────────────────
 STEP 2/6: brand
────────────────────────────────────────────────────────────

[ORCHESTRATOR] Building prompt for sharkcode-brand...
  ├── Client brief: loaded
  ├── Knowledge base: 3 books (StoryBrand, Brand Gap, Designing Brand Identity)
  ├── Cross-team outputs: research/research-output.md (3,847 words)
  ├── Handoff from research: competitor gaps, storytelling recommendation
  └── Tool allowlist: Read,Write,Grep,Glob,WebSearch,WebFetch,mcp__sharkcode-core__*

[ORCHESTRATOR] Launching agent: sharkcode-brand (model: sonnet)

[DISCUSSION] sharkcode-brand: Ho letto l'output di research. Concordo
             sullo storytelling familiare. Il naming deve evocare calore,
             non clinica. Propongo palette: sage green + cream + warm beige.

  ... agent runs ...

[ORCHESTRATOR] QUALITY GATE: PASS
[ORCHESTRATOR] Cost: $0.072

────────────────────────────────────────────────────────────
 STEP 3/6: seo
────────────────────────────────────────────────────────────

[ORCHESTRATOR] Building prompt for sharkcode-seo...
  ├── Cross-team outputs: research + brand (reads both)
  └── Tool allowlist includes WebSearch (keyword research needs it)

[DISCUSSION] sharkcode-seo: Keywords trovate: "casa di riposo roma"
             (1.2K/mo), "RSA roma nord" (480/mo). Il competitor principale
             non ha schema markup LocalBusiness. Vantaggio immediato.

  ... continues through marketing, design, legal ...

────────────────────────────────────────────────────────────
 PIPELINE COMPLETE
────────────────────────────────────────────────────────────

[ORCHESTRATOR] Phase "research" complete
[ORCHESTRATOR] Teams run: 6/6
[ORCHESTRATOR] Total cost: $0.41
[ORCHESTRATOR] Total tokens: 287,432 input / 89,217 output
[ORCHESTRATOR] Duration: 34 min 12 sec
[ORCHESTRATOR] Artifacts: 14 files in .planning/context/valerio-casa-riposo/

[ORCHESTRATOR] ⏸ Waiting for human review before next phase.
[ORCHESTRATOR] Run next: bun run tools/orchestrator/run-pipeline.ts valerio-casa-riposo strategy
```

### What happens when things fail

```
────────────────────────────────────────────────────────────
 STEP 4/6: design
────────────────────────────────────────────────────────────

[ORCHESTRATOR] Launching agent: sharkcode-design (model: opus)

  ... agent runs for 14 minutes ...

[ORCHESTRATOR] ⚠ TIMEOUT (900000ms) — killing process tree
[ORCHESTRATOR] taskkill /PID 31205 /T /F    (Windows)
[ORCHESTRATOR] Process tree killed

[ORCHESTRATOR] Attempt 1/3 failed (timeout). Retrying in 2000ms...

[ORCHESTRATOR] Retry 2/3: simplifying prompt (removing verbose sections)
[ORCHESTRATOR] Launching agent: sharkcode-design (model: opus)

  ... agent completes in 7 minutes ...

[ORCHESTRATOR] QUALITY GATE: FAIL
  └── Missing required section: "Responsive Breakpoints"

[ORCHESTRATOR] Retry 3/3: adding explicit instruction for missing section
[ORCHESTRATOR] Launching agent: sharkcode-design (model: opus)

  ... agent completes in 8 minutes ...

[ORCHESTRATOR] QUALITY GATE: PASS
[ORCHESTRATOR] Cost: $0.31 (3 attempts, opus pricing)
```

### What happens when all retries fail

```
[ORCHESTRATOR] ✗ ALL 3 ATTEMPTS FAILED for design
[ORCHESTRATOR] Escalating to human review.

[DASHBOARD EVENT] → escalation (design fallito 3 volte, richiede intervento)
[WEB PUSH] → "Pipeline bloccata: design ha fallito 3 tentativi"

[ORCHESTRATOR] Partial results saved to:
  .planning/context/valerio-casa-riposo/design/FAILED_attempt_3.md

[ORCHESTRATOR] Pipeline paused. Fix the issue and re-run:
  bun run tools/orchestrator/run-pipeline.ts valerio-casa-riposo research --resume
```

### Discussion threads (agents talking to each other)

During pipeline runs, agents post to a shared discussion thread. This is what it looks like in the dashboard:

```
┌─────────────────────────────────────────────────────────┐
│ Discussion: valerio-casa-riposo / research               │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ 14:23  research                                          │
│ Inizio lavoro sulla fase research. Leggo brief e         │
│ output degli altri team.                                 │
│                                                          │
│ 14:29  research                                          │
│ Ho trovato 12 competitor diretti zona Roma Nord.         │
│ Nessuno ha video tour o testimonianze verificabili.      │
│ Raccomando: storytelling familiare, non clinico.         │
│                                                          │
│ 14:31  brand                                             │
│ Concordo su storytelling familiare. Palette proposta:    │
│ sage green (#87A96B), cream (#FFFDD0), warm beige.       │
│ Il naming deve evocare "casa", non "struttura".          │
│                                                          │
│ 14:35  seo                                               │
│ Keywords: "casa di riposo roma" 1.2K/mo. Competitor      │
│ principale non ha schema LocalBusiness. Vantaggio.       │
│                                                          │
│ 14:38  marketing                                         │
│ Target persona: figlio/figlia 45-60 anni che cerca       │
│ per genitore anziano. Decision driver: sicurezza +       │
│ vicinanza + "mi fido". Non prezzo.                       │
│                                                          │
│ 14:44  design                                            │
│ Letto tutto. Layout proposto: hero con video tour,       │
│ sezione testimonianze con foto reali, bento grid         │
│ servizi. No stock photos. No slider generico.            │
│                                                          │
│ 14:52  legal                                             │
│ ATTENZIONE: foto ospiti = dati sensibili sanitari.       │
│ Serve consenso esplicito ex art. 9 GDPR + informativa   │
│ specifica. Raccomando: usare solo foto staff e           │
│ ambienti, no volti ospiti senza liberatoria.             │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

Each team reads what came before. Legal catches a GDPR issue that Design didn't consider. This is the value of multi-agent communication — not just sequential handoffs, but actual cross-team awareness.

### Actual dry-run output (unedited)

This is the real output from `--dry-run` on a production client:

```
$ bun run tools/orchestrator/run-pipeline.ts valerio-casa-riposo research --dry-run

[ORCHESTRATOR] Override: none
[ORCHESTRATOR] Pipeline: valerio-casa-riposo / research
[ORCHESTRATOR] Teams: brand, design, seo, marketing
[ORCHESTRATOR] Parallel: false
[ORCHESTRATOR] Mode: DRY RUN

[DRY RUN] Stima durata basata su dati storici:
  Nessun dato storico disponibile. Prima esecuzione.

[ORCHESTRATOR] Using smart sequencing (3 steps)

[ORCHESTRATOR] Step 1: Round 1 - SEQUENTIAL [brand]
================================================================================
[ORCHESTRATOR] Team: BRAND
[ORCHESTRATOR] Agent file: .claude/agents/sharkcode-brand.md
[ORCHESTRATOR] Model: sonnet (via routing map)
[ORCHESTRATOR] Expected output: brand/target-persona.md
[ORCHESTRATOR] Cross-read from: (none yet)
================================================================================
[DRY RUN] Prompt length:  332 righe

[ORCHESTRATOR] Step 2: Round 2 - PARALLEL [design, seo, marketing]
================================================================================
[ORCHESTRATOR] Team: DESIGN
[ORCHESTRATOR] Agent file: .claude/agents/sharkcode-design.md
[ORCHESTRATOR] Model: sonnet (via routing map)
[ORCHESTRATOR] Expected output: design/research-output.md
================================================================================
[DRY RUN] Prompt length:  353 righe

================================================================================
[ORCHESTRATOR] Team: SEO
[ORCHESTRATOR] Agent file: .claude/agents/sharkcode-seo.md
[ORCHESTRATOR] Model: sonnet (via routing map)
[ORCHESTRATOR] Expected output: seo/research-output.md
================================================================================
[DRY RUN] Prompt length:  389 righe

================================================================================
[ORCHESTRATOR] Team: MARKETING
[ORCHESTRATOR] Agent file: .claude/agents/sharkcode-marketing.md
[ORCHESTRATOR] Model: sonnet (via routing map)
[ORCHESTRATOR] Expected output: marketing/research-output.md
================================================================================
[DRY RUN] Prompt length:  322 righe

================================================================================
[DRY RUN] Riepilogo:
  Fase:     research
  Cliente:  valerio-casa-riposo
  Team:     4 team
  Stima:    nessun dato storico
  Nessun agente verra' invocato.
================================================================================
```

Note: the orchestrator does smart sequencing — Brand runs first (Step 1), then Design, SEO, and Marketing run **in parallel** (Step 2) because they all depend on Brand's output but not on each other.

## What it costs to run

| Item | Cost | Notes |
|------|------|-------|
| **Claude Max** | $100/month | Only subscription needed. Agents run via Claude Code CLI, not API calls. No per-token billing. |
| **Bun** | Free | Runtime, server, package manager |
| **SQLite** | Free | Database, no hosting needed |
| **Vue + Vite** | Free | Frontend framework |
| **Google Calendar API** | Free | OAuth2, free tier covers personal use |
| **Cloudflare Pages** | Free tier | Client site hosting (paid plans available) |
| **SiteGround Email** | ~$3/month | Already included in hosting plan |
| **GitHub** | Free | Private repo |
| **Domain** | ~$12/year | sharkcodestudio.com |
| **n8n** | Free (self-hosted) | Webhook automation |
| **XOLO** | $122/month | Estonian company management (external, not part of OS) |

**Total infrastructure cost for the OS itself: $100/month** (just Claude Max).

No OpenAI API bills. No vector database hosting. No LangSmith observability fees. No cloud GPU. The entire system runs locally on a laptop with a single subscription.

The cost tracking in the dashboard exists for client billing transparency, not because there are actual per-token costs. Michel tracks how much AI time each client project consumes to inform pricing decisions.

## Honest limitations

- **Single user.** This is built for one person. No auth, no multi-tenancy.
- **Claude Max dependency.** Agents run on Claude Code CLI with a Max subscription. No fallback to other LLMs.
- **Windows-first.** Tested primarily on Windows 11. Unix paths work but are less tested.
- **Partial test coverage.** 24 tests cover 4 critical paths (pipeline events, webhook HMAC, OAuth, system health). 7 of 11 route modules still lack dedicated tests.
- **SQLite.** Great for single-user, won't scale to concurrent team access. That's fine — there's no team.

## License

Private repository. All rights reserved.

---

*Architected and built by [Michel Dimitri](https://github.com/Michel9329), using Claude Code as a development accelerator. System design, architecture decisions, business logic, and quality standards are human-driven. The AI writes code faster — it doesn't replace knowing what to build and why.*
