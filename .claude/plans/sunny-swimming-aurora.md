# Plan: tau-platform Repository Conventions & Initial Discussions

## Goal

Establish repository conventions for `tau-platform` and publish 6 initial GitHub Discussions (1 announcement + 5 topic discussions) to kick off organizational standards alignment.

---

## Step 1: Repository Directory Structure

Restructure `tau-platform` to enforce the convention: **visual artifacts in `planning/`, text-based artifacts as discussion drafts in `drafts/`, formalized standards in `standards/`**.

```
tau-platform/
├── README.md                          # Repository conventions and lifecycle
├── drafts/                            # Discussion drafts (markdown)
│   ├── 00-initial-alignment.md        # Announcement draft
│   ├── 01-ontology.md
│   ├── 02-project-management.md
│   ├── 03-iterative-development.md
│   ├── 04-web-service-architecture.md
│   └── 05-claude-skills.md
├── standards/                         # Formalized standard documents
│   └── (empty — populated as discussions conclude)
├── planning/                          # Visual artifacts (diagrams, wireframes)
│   ├── long-term-diagram.png
│   ├── long-term-diagram-with-models.png
│   └── phase-diagram.png
└── archive/                           # Historical pre-convention artifacts
    ├── ECOSYSTEM_VISION.md
    ├── discussion framework-brainstorm.txt
    └── web-service-template.txt
```

**Actions:**
- Create directories: `drafts/`, `standards/`, `planning/`, `archive/`
- Move `*.png` → `planning/`
- Move `ECOSYSTEM_VISION.md`, `*.txt` → `archive/`
- Rewrite `README.md` with repository conventions (see Step 2)

**Files modified:** `~/tau/tau-platform/README.md` (rewrite)
**Files moved:** 5 existing files into new directories

---

## Step 2: README — Repository Conventions

Rewrite `README.md` to document:

1. **Purpose** — Organizational coordination, planning, and standards for the TAU Platform ecosystem
2. **Directory conventions** — What goes where (`drafts/`, `standards/`, `planning/`, `archive/`)
3. **Standards lifecycle**:
   - **Draft** → Write discussion content in `drafts/` as markdown
   - **Discuss** → Publish as a GitHub Discussion; team reviews and iterates
   - **Formalize** → Extract resolved decisions into `standards/` documents
   - **Operationalize** → Create or update Claude Code skills to encode the standard
4. **Discussion categories** — Announcements, Architecture, Ideas, Q&A
5. **Naming conventions** — Draft files: `NN-kebab-case.md`, Standards: `kebab-case.md`

**File:** `~/tau/tau-platform/README.md`

---

## Step 3: Write Discussion Drafts

Write 6 markdown files in `drafts/`. Each draft contains the full discussion body ready to publish. Content sources are noted for each.

### 3.0 — `00-initial-alignment.md` (Announcement)

**Title:** TAU Platform: Initial Standards & Alignment
**Category:** Announcements

Content:
- Brief TAU Platform vision (distilled from `ECOSYSTEM_VISION.md`)
- Purpose of this discussion series — establish foundational standards
- Standards lifecycle explanation (draft → discuss → formalize → operationalize)
- Links to each of the 5 topic discussions (added after creation)
- Call to action: review and contribute to each topic

### 3.1 — `01-ontology.md`

**Title:** Ontology: Common Vocabulary for the TAU Platform
**Category:** Architecture

Content:
- Why a shared vocabulary matters for cross-repo development
- Term definitions organized by domain:
  - **Platform layer**: TAU (Tailored Agentic Units), Ecosystem, Library, Service
  - **Agent primitives** (from tau-core): Agent, Protocol, Provider, Model, Client
  - **Orchestration** (from go-agents-orchestration): Hub, State Graph, Workflow, Chain, Message
  - **Data & RAG** (from ECOSYSTEM_VISION.md): Chunk, Entity, Embedding, Retrieval, Context
  - **Infrastructure** (from agent-lab): Module, System, Handler, Repository, Lifecycle
  - **Development** (from Claude skills): Skill, Standard, Draft, Phase, Target
- Terms flagged from `discussion framework-brainstorm.txt`: Agent vs. Tool vs. Skill vs. Function vs. Service vs. Hub vs. Unit — need disambiguation
- Open questions for team alignment

**Sources:** `archive/discussion framework-brainstorm.txt` (nomenclature section), tau-core `pkg/` package names, go-agents-orchestration README, ECOSYSTEM_VISION.md

### 3.2 — `02-project-management.md`

**Title:** Project Management: Infrastructure & Processes
**Category:** Architecture

Content:
- GitHub Projects v2 with phase-based tracking (already initialized)
- Three-tier hierarchy: Project → Phase (SINGLE_SELECT field) → Item (issue/PR/draft)
- Milestone convention: each non-meta phase gets a corresponding milestone on linked repos
- Cross-repo backlog: all ecosystem repos linked to the "TAU Platform" project
- Issue conventions: labels, templates, sizing
- Claude Code skill: `project-management` — describe what it automates and how to use it
- Discussion as a coordination layer (this repo's role)

**Sources:** `project-management` skill SKILL.md, existing GitHub Project setup

### 3.3 — `03-iterative-development.md`

**Title:** Iterative Development: A Kaizen Approach to Architecture
**Category:** Architecture

Content:
- Core philosophy: build from the ground up, introduce complexity at the moment it's needed
- **Target system**:
  - **20m targets** — Critical, immediate focus items. What we're building now.
  - **300m targets** — Directional goals we're working toward. Inform decisions but don't dictate implementation.
- Big-picture planning is valuable for anticipation, not for locking in architecture
- Avoid analysis paralysis: start with what we have, iterate
- Kaizen mindset applied to software: small, continuous improvements compound
- Practical examples from TAU ecosystem:
  - tau-core started as go-agents, evolved through real usage
  - Protocols added one at a time (Chat → Vision → Tools → Embeddings → Audio next)
  - Providers added as needed (Ollama → Azure → Whisper next)
- How this interacts with project management: phases as iteration boundaries

### 3.4 — `04-web-service-architecture.md`

**Title:** Web Service Architecture: Go + Fiber Standards
**Category:** Architecture

Content:
- Starting point: `archive/web-service-template.txt` directory layout
- Reference implementation: `agent-lab` architecture patterns
- **Configuration lifecycle** (adopt from agent-lab):
  - Layered config: base file + environment overlay + env var overrides
  - `Load() → Merge() → Finalize()` pattern
  - Config is ephemeral — transformed at boundaries, discarded after initialization
  - Each subsystem has Config struct, Env map, Finalize() method
- **Cold start + hot start**:
  - Cold: load config → create infrastructure → create domain systems → wire modules
  - Hot: start lifecycle → register hooks → serve
- **Modular sub-systems**:
  - Infrastructure layer (shared): lifecycle coordinator, logger, database, storage
  - Domain systems (isolated): System interface → Handler → Routes
  - Module router: prefix-based routing to isolated modules (API, app, docs)
- **Server-managed infrastructure**:
  - Lifecycle coordinator: `OnStartup()`, `OnShutdown()`, `WaitForStartup()`, context cancellation
  - Health endpoints: `/healthz` (liveness), `/readyz` (startup-gated readiness)
  - Graceful shutdown: SIGTERM → cancel context → wait for hooks with timeout
- **Domain system pattern**:
  - Interface defines business operations
  - Handler translates HTTP ↔ domain calls
  - Routes declare endpoints with method, pattern, handler
  - Repository encapsulates data access
  - Cross-domain: runtime aggregation (domains reference other domain interfaces)
- **Mapping to Fiber**:
  - `fiber.App` replaces custom module router
  - `fiber.Router` / `app.Group()` for module isolation
  - Fiber middleware replaces custom middleware stack
  - Fiber `Shutdown()` integrated with lifecycle coordinator
  - Standard library patterns (interfaces, structs) remain unchanged
- Open questions: middleware selection, error handling conventions, OpenAPI generation in Fiber

**Sources:** `archive/web-service-template.txt`, agent-lab `cmd/server/`, `pkg/`, `internal/`

### 3.5 — `05-claude-skills.md`

**Title:** Claude Code Skills: AI-Assisted Development Standards
**Category:** Architecture

Content:
- What Claude Code skills are and how they work (trigger-based, automatic invocation)
- Skills established in tau-core (6 skills):
  - `go-patterns` — Go design principles, interfaces, error handling
  - `github-cli` — GitHub CLI operations with 11 reference files
  - `project-management` — GitHub Projects v2, phases, cross-repo backlogs
  - `skill-creator` — Meta-skill for creating new skills
  - `tau-core-admin` — Contributing to tau-core internals
  - `tau-core-dev` — Building applications with tau-core
- Skill anatomy: SKILL.md frontmatter (name, description, triggers) + content + optional `references/` directory
- Project settings: `.claude/settings.json` for plans directory, allow permissions
- Session continuity: plan files in `.claude/plans/` with context snapshots
- Vision: every formalized standard should have a corresponding skill
- How skills compound: each new standard encoded as a skill makes the AI agent progressively more capable within the ecosystem

**Sources:** tau-core `.claude/skills/*/SKILL.md`, `.claude/settings.json`, `.claude/CLAUDE.md`

---

## Verification

- [ ] `tau-platform` directory structure matches convention (`drafts/`, `standards/`, `planning/`, `archive/`)
- [ ] Existing files reorganized into correct directories
- [ ] `README.md` documents conventions and lifecycle
- [ ] 6 draft files exist in `drafts/`
- [ ] Drafts reviewed and approved by user before publishing
