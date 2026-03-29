# Composable AI Skills Framework

A framework for building Claude AI skills that compose — where deep domain knowledge
and task-specific capabilities snap together like building blocks to produce
grounded, citable, expert-quality responses.

---

## The problem this solves

Generic AI assistants know a little about everything. That is useful for general
questions. It is not useful when you need Claude to behave like a credentialed
expert in a specific system — one who knows which version you are running, where
your documentation lives, what the compliance implications of a configuration
choice are, and how to produce a document that will survive regulatory scrutiny.

This framework solves that by giving Claude two things it does not have by default:

**Deep product knowledge** — a skill that knows a specific system in detail:
its architecture, its version history, its compliance-critical configuration
points, its known failure modes, and exactly where to find authoritative
information in your documentation store.

**Task capability** — a skill that knows how to do something with that knowledge:
evaluate products against requirements, co-author compliance documents,
triage incidents, prepare for audits.

Load one of each. They compose automatically through a shared context protocol.
The product skill sets the table. The capability skill does the work.

---

## The core idea — two tiers, one context

```
┌─────────────────────────────────────────────────────┐
│  Tier 1 — Product skill                             │
│  Deep knowledge of a specific system                │
│  Passive — establishes context, does not act alone  │
└────────────────────┬────────────────────────────────┘
                     │ writes context block
                     ▼
┌─────────────────────────────────────────────────────┐
│  Shared session context                             │
│  Product · Version · Document paths · KB URLs       │
│  Domain guardrails · Archetype · Session date       │
└────────────────────┬────────────────────────────────┘
                     │ reads context block
                     ▼
┌─────────────────────────────────────────────────────┐
│  Tier 2 — Capability skill                          │
│  Knows how to do something with product knowledge   │
│  Active — retrieves, synthesises, produces output   │
└─────────────────────────────────────────────────────┘
```

The product skill and capability skill never need to be designed together.
Any product skill works with any compatible capability skill. That is the
composability guarantee — and it is enforced by a shared context protocol
that every skill in this framework must implement.

---

## How skills compose — the Lego model

Think of skills as Lego blocks. Each block has a standard connector — the
context protocol — so any block can attach to any other. The blocks themselves
can be anything: a specific software system, a document authoring capability,
an incident triage workflow, a compliance audit process.

A user assembles the blocks they need for their task:

```
[Product skill A] + [Capability skill X]
→ Deep product knowledge applied to task X

[Product skill A] + [Product skill B] + [Capability skill Y]
→ Two products evaluated side by side against task Y

[Product skill A] + [Capability skill X] → output → [Capability skill Z]
→ Sequential workflow: X produces output that Z consumes
```

The user always decides which blocks to load. Skills never autonomously chain.
Every transition is a conscious human action. This is a deliberate design
choice — not a limitation.

---

## What a skill is

A skill is a single markdown file: `SKILL.md`.

It contains two zones:

**Fixed zone** — authored by the skill builder, reviewed by maintainers,
never edited by end users. Contains the skill's logic, knowledge, retrieval
instructions, guardrails, and output formats.

**Config zone** — a small, clearly delimited block of placeholder variables
that the end user fills in with their own environment details: document store
paths, knowledge base URLs, system-specific settings.

```
## ── CONFIGURE THIS SECTION — YOUR ENVIRONMENT ONLY ──

KNOWLEDGE_PATH:   [path to your document store]
KB_URL:           [your knowledge base URL]

## ── END CONFIGURATION — DO NOT EDIT BELOW THIS LINE ──
```

The user fills in the config zone once. The skill logic never changes.
Their environment details never leave their environment — they are never
committed to the repository.

---

## What a skill is not

A skill is not a plugin, an API, or a piece of software. It does not execute
code. It does not call other skills directly. It does not store data between
sessions.

A skill is a precise set of instructions written in plain English that tells
Claude how to behave when that skill is loaded. Claude reads the instructions
and follows them. The sophistication is in the instructions — how precisely
they define retrieval, synthesis, citation, guardrails, and handoff.

This means anyone who can write clearly can build a skill. Domain expertise
matters more than technical skill. A validation engineer who knows Empower
deeply can build a better Empower product skill than a developer who does not.

---

## Repository structure

```
composable-ai-skills/
│
├── framework/                        ← domain-agnostic core — start here
│   ├── README.md                     ← this file
│   ├── SKILL_BUILDING_GUIDE.md       ← how to build a skill (full guide)
│   ├── CONTEXT_PROTOCOL.md           ← the shared context specification
│   ├── GUARDRAIL_FRAMEWORK.md        ← configurable compliance rules
│   ├── templates/
│   │   ├── PRODUCT_SKILL_TEMPLATE.md
│   │   └── CAPABILITY_SKILL_TEMPLATE.md
│   └── examples/
│       └── minimal-example/          ← simplest possible working skill pair
│
├── implementations/                  ← domain-specific skill libraries
│   ├── gxp-lab-automation/           ← reference implementation
│   │   ├── README.md
│   │   ├── product-skills/
│   │   │   └── waters-empower/
│   │   │       ├── SKILL.md
│   │   │       └── CHANGELOG.md
│   │   └── capability-skills/
│   │       └── gxp-sdlc-coauthor/
│   │           ├── SKILL.md
│   │           └── CHANGELOG.md
│   └── [your-domain]/                ← contribute your implementation here
│
├── CONTRIBUTING.md
├── LICENSE
└── .gitignore
```

---

## Who builds what

**Framework contributors** — improve the core: templates, protocol spec,
guardrail framework, building guide. Changes here require careful review
because they affect every skill built on the framework.

**Implementation contributors** — build skills for a specific domain.
They use the framework but do not change it. A validation engineer
building a LIMS skill, a consultant building an ERP skill — these are
implementation contributors.

**End users** — load skills into their Claude environment, fill in the
config zone with their own paths and URLs, and use the resulting
composed expert assistant. They do not need to understand the framework
to use the skills.

---

## What makes this framework different

**Grounded responses.** Every claim in a skill response traces to a
retrieved document or knowledge base article. The framework enforces
citation by design — it is not optional for skill builders.

**Composability without coupling.** Product skills and capability skills
are built independently by different contributors who may never meet.
They compose correctly because the context protocol is precise and
mandatory — not because the builders coordinated.

**Domain-configurable compliance.** The guardrail framework adapts to
the compliance requirements of any regulated domain. GxP, SOX, ISO 13485,
HIPAA — each domain configures the same guardrail slots with its own rules.
The enforcement mechanism is identical across domains.

**Human in the loop.** Skills suggest. Humans decide. No autonomous
chaining. This is the correct design for any domain where actions need
a human decision and an audit trail behind them.

**Plain text, version-controlled.** Every skill is a markdown file.
Every change is a commit. Every version is a tag. `git diff` on a
SKILL.md shows exactly what changed in the retrieval protocol between
versions. This is rare for knowledge assets and essential for regulated
environments.

---

## The reference implementation

The `implementations/gxp-lab-automation/` folder is the reference
implementation — a complete, production-quality skill library for
lab automation systems (LIMS, CDS, SDMS, ELN) in GxP-regulated
pharmaceutical environments.

It demonstrates every framework concept in a real, complex, regulated
domain. Read it to understand what a mature implementation looks like.
Use the framework documents to understand why it is structured the way
it is.

The reference implementation is a proof of the framework, not the
definition of it. The framework applies equally to any domain where
Claude needs deep, grounded, version-specific, compliance-aware
expertise.

---

## Where to start

**If you want to use existing skills:**
Go to `implementations/` → find your domain → read the implementation
README → follow setup instructions → fill in your config zone.

**If you want to build a new product skill:**
Read `framework/SKILL_BUILDING_GUIDE.md` → start from
`framework/templates/PRODUCT_SKILL_TEMPLATE.md` → study the reference
implementation for examples.

**If you want to build a new capability skill:**
Read `framework/SKILL_BUILDING_GUIDE.md` → read
`framework/CONTEXT_PROTOCOL.md` → start from
`framework/templates/CAPABILITY_SKILL_TEMPLATE.md`.

**If you want to start a new domain implementation:**
Read all four framework documents → create a folder under
`implementations/[your-domain]/` → start with one product skill
and one capability skill → validate the handoff works before building more.

---

## Licence and legal

MIT licence. See `LICENSE`.

Product names used in skill files are used solely to identify the systems
those skills support. This project is independent and community-maintained.
It is not affiliated with, endorsed by, or sponsored by any vendor whose
products are referenced in implementation skill files.

Users are responsible for ensuring their use of vendor documentation
with these skills is consistent with their licence agreements with
the respective vendors. The framework repository contains no vendor
documentation. See `CONTRIBUTING.md` for full legal and IP guidance.
