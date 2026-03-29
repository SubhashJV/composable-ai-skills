# Composable AI Skills Framework

A framework for building Claude AI skills that compose — where deep domain
knowledge and task-specific capabilities snap together like building blocks
to produce grounded, citable, expert-quality responses in any complex or
regulated domain.

> **Status:** Framework live · Reference implementation in development · Skills coming soon

---

## The problem this solves

Generic AI assistants know a little about everything. That is useful for
general questions. It is not useful when you need Claude to behave like a
credentialed expert in a specific system — one who knows which version you
are running, where your documentation lives, what the compliance implications
of a configuration choice are, and how to produce output that will survive
professional or regulatory scrutiny.

This framework solves that by giving Claude two things it does not have by default:

- **Deep system knowledge** — grounded in your own documentation, version-specific,
  retrieved and cited rather than recalled from training
- **Task capability** — structured workflows for evaluation, document authoring,
  incident triage, audit preparation, and more

---

## How it works — the Lego model

Two skill types. One context protocol. Infinite compositions.

```
┌──────────────────────────────────┐
│  Tier 1 — Product skill          │
│  Deep knowledge of a system      │
│  Passive — establishes context   │
└─────────────────┬────────────────┘
                  │ writes context block
                  ▼
┌──────────────────────────────────┐
│  Shared session context          │
│  System · Version · Doc paths    │
│  KB URLs · Domain · Guardrails   │
└─────────────────┬────────────────┘
                  │ reads context block
                  ▼
┌──────────────────────────────────┐
│  Tier 2 — Capability skill       │
│  Knows how to do something       │
│  Active — retrieves, synthesises │
│  produces grounded output        │
└──────────────────────────────────┘
```

Any product skill works with any compatible capability skill — because every
skill implements the same shared context protocol. Adding one new product skill
makes it immediately work with every existing capability skill. Adding one new
capability skill makes it immediately work with every existing product skill.

### Skill compositions

```
[Product skill A] + [Capability skill X]
→ Deep system knowledge applied to task X

[Product skill A] + [Product skill B] + [Capability skill Y]
→ Two systems evaluated side-by-side against task Y

[Product skill A] + [Capability X] → output → [Capability Z]
→ Sequential workflow: X produces output that Z consumes
```

The user always decides which skills to load and when. Skills never chain
autonomously — every transition is a conscious human decision.

---

## What a skill is

A skill is a single markdown file — `SKILL.md` — containing precise
instructions that tell Claude how to behave when that skill is loaded.

Every skill has two zones:

**Fixed zone** — authored by the skill builder, reviewed before publishing,
never edited by end users. Contains the skill's knowledge, retrieval logic,
guardrails, and output formats.

**Config zone** — a small, clearly delimited block the end user fills in
with their own environment details: document store paths, knowledge base URLs.

```
## ── CONFIGURE THIS SECTION — YOUR ENVIRONMENT ONLY ──

KNOWLEDGE_PATH:   [path to your document store]
KB_URL:           [your knowledge base URL]

## ── END CONFIGURATION — DO NOT EDIT BELOW THIS LINE ──
```

The user fills in the config zone once. The skill logic never changes.
Their environment details never leave their environment.

---

## Knowledge sources

Skills retrieve from two sources the user configures:

**Document store** — a SharePoint folder, shared drive, or equivalent
containing documentation organised by system and version. Skills read
from paths the user configures. No documentation is included in this
repository — users supply their own from their own licensed sources.

**Live knowledge base** — vendor or domain knowledge base URLs registered
in the skill. Fetched at query time. Every article retrieved is date-stamped.
Stale content is flagged automatically.

---

## Guardrails — enforced by design

Every skill implements seven non-negotiable guardrails. These apply across
all domains and are reviewed on every pull request:

| Guardrail | What it enforces |
|---|---|
| **Source discipline** | Claims grounded only in retrieved sources — no general knowledge substitution |
| **Fabrication firewall** | Unverifiable content marked `[UNVERIFIED]` — never stated as fact |
| **Version discipline** | Retrieval pinned to the confirmed system version — no cross-version contamination |
| **Conflict surfacing** | Contradictions between sources shown to user — never silently resolved |
| **Domain safety filter** | Content that bypasses domain integrity controls flagged as unsafe |
| **Freshness signalling** | Retrieved content older than threshold flagged with date |
| **Human decision gate** | No autonomous chaining — user confirms every skill transition |

---

## Repository structure

```
composable-ai-skills/
│
├── framework/                        ← domain-agnostic core — start here
│   ├── README.md                     ← framework concepts and design
│   ├── SKILL_BUILDING_GUIDE.md       ← complete how-to for skill builders
│   ├── CONTEXT_PROTOCOL.md           ← shared context specification (normative)
│   ├── GUARDRAIL_FRAMEWORK.md        ← seven configurable compliance guardrail slots
│   ├── templates/
│   │   ├── PRODUCT_SKILL_TEMPLATE.md ← fill-in-the-blanks for product skills
│   │   └── CAPABILITY_SKILL_TEMPLATE.md ← fill-in-the-blanks for capability skills
│   └── examples/
│       └── minimal-example/          ← simplest working skill pair
│
├── implementations/                  ← domain-specific skill libraries
│   └── [domain-name]/                ← one folder per domain
│       ├── README.md
│       ├── product-skills/           ← one folder per system
│       ├── capability-skills/        ← one folder per capability
│       └── guardrail-profiles/       ← domain compliance configuration
│
├── CONTRIBUTING.md                   ← authoring guide + legal and IP guidance
└── LICENSE                           ← MIT
```

---

## Framework documents

| Document | Purpose |
|---|---|
| [SKILL_BUILDING_GUIDE.md](framework/SKILL_BUILDING_GUIDE.md) | Complete step-by-step guide to building any skill |
| [CONTEXT_PROTOCOL.md](framework/CONTEXT_PROTOCOL.md) | Exact specification for context blocks and handoff signals |
| [GUARDRAIL_FRAMEWORK.md](framework/GUARDRAIL_FRAMEWORK.md) | Seven guardrail slots with domain-configurable content |
| [PRODUCT_SKILL_TEMPLATE.md](framework/templates/PRODUCT_SKILL_TEMPLATE.md) | Starting point for product skills |
| [CAPABILITY_SKILL_TEMPLATE.md](framework/templates/CAPABILITY_SKILL_TEMPLATE.md) | Starting point for capability skills |

---

## Implementations

An implementation is a collection of product skills and capability skills
for a specific domain, built on this framework. Each lives in its own folder
under `implementations/` with its own README, guardrail profile, and skill set.

### Current implementations

| Implementation | Domain | Status |
|---|---|---|
| [gxp-lab-automation](implementations/gxp-lab-automation/) | Lab automation systems in regulated environments | In development |

### Starting your own implementation

Any domain where Claude needs deep, grounded, version-specific expertise
is a candidate — clinical trial systems, ERP platforms, quality management
systems, financial compliance systems, manufacturing execution systems,
and more.

Read all four framework documents → create a folder under
`implementations/[your-domain]/` → start with one product skill and one
capability skill → validate the handoff before building more.

---

## Who builds what

**Framework contributors** — improve the core: templates, protocol spec,
guardrail framework, building guide. Changes here affect every skill built
on the framework and require careful review.

**Implementation contributors** — build skills for a specific domain using
the framework. Domain expertise matters more than technical skill. The person
who knows a system deeply builds the best skill for it.

**End users** — load skills into Claude, fill in the config zone with their
own paths and URLs, and use the composed expert assistant. No framework
knowledge required to use skills.

---

## Where to start

**Use existing skills** → go to `implementations/` → find your domain →
read the implementation README → follow the setup instructions.

**Build a product skill** → read the [Skill Building Guide](framework/SKILL_BUILDING_GUIDE.md)
→ start from the [Product Skill Template](framework/templates/PRODUCT_SKILL_TEMPLATE.md).

**Build a capability skill** → read the [Skill Building Guide](framework/SKILL_BUILDING_GUIDE.md)
and [Context Protocol](framework/CONTEXT_PROTOCOL.md)
→ start from the [Capability Skill Template](framework/templates/CAPABILITY_SKILL_TEMPLATE.md).

**Start a new domain implementation** → read all four framework documents
→ open a discussion so the community can help.

**Report an issue or contribute** → read [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Legal

MIT licence. See [LICENSE](LICENSE).

System names used in implementation skill files are used solely to identify
the systems those skills support. This project is independent and
community-maintained. It is not affiliated with, endorsed by, or sponsored
by any vendor or organisation referenced in implementation files.

Users are responsible for ensuring their use of third-party documentation
with these skills is consistent with their licence agreements. This
repository contains no third-party documentation. See
[CONTRIBUTING.md](CONTRIBUTING.md) for full legal and IP guidance.

---

## Acknowledgements

Built on the idea that domain expertise — not engineering skill — is what
makes an AI assistant genuinely useful in a complex or regulated environment.
Every skill contributed by someone who knows a system deeply makes Claude
more useful for everyone who works with that system.
