# Composable AI Skills Framework

A framework for building Claude AI skills that compose — where deep domain
knowledge and task-specific capabilities snap together like building blocks
to produce grounded, citable, expert-quality responses.

> **Status:** Framework live · Reference implementation in development · Skills coming soon

---

## What this is

Generic AI assistants know a little about everything. That is useful for
general questions. It is not useful when you need Claude to behave like a
credentialed expert in a specific system — one who knows which version you
are running, where your documentation lives, what the compliance implications
of a configuration choice are, and how to produce a document that will
survive regulatory scrutiny.

This framework solves that through two composable skill types:

| Skill type | What it does |
|---|---|
| **Product skill** (Tier 1) | Deep knowledge of a specific system — architecture, versions, compliance-critical config, known failure modes, documentation retrieval |
| **Capability skill** (Tier 2) | Knows how to do something with that knowledge — evaluate, co-author documents, triage incidents, prepare for audits |

Load one of each. They compose automatically through a shared context
protocol. The product skill sets the table. The capability skill does the work.

---

## How it works — the Lego model

```
┌─────────────────────────────────┐     ┌─────────────────────────────────┐
│  Product skill                  │     │  Capability skill               │
│  e.g. waters-empower            │     │  e.g. gxp-sdlc-coauthor         │
│                                 │     │                                 │
│  Knows: Empower architecture    │     │  Does: co-authors IQ/OQ/PQ,     │
│  versions, Part 11 config,      │ ──► │  URS, change control docs       │
│  your SharePoint paths,         │     │  grounded in retrieved docs,    │
│  vendor KB URLs                 │     │  with full citation trail       │
└─────────────────────────────────┘     └─────────────────────────────────┘
```

Any product skill works with any compatible capability skill.
That is the composability guarantee — enforced by a shared context protocol
every skill must implement.

### Example — writing an OQ protocol for a Waters Empower upgrade

1. Load `waters-empower` skill → Claude confirms version, reads your SharePoint path, connects to vendor KB
2. Load `gxp-sdlc-coauthor` skill → reads Empower context automatically, no repeated questions
3. Claude generates OQ test cases sourced directly from your configured documentation
4. Every test case cited to its source document and section
5. Gaps flagged with `[UNVERIFIED]` where documentation does not cover a required area

### Example — evaluating three LIMS products against a URS

1. Load `labware-lims` + `labvantage-lims` + `starlims` simultaneously
2. Load `product-evaluation` skill → reads all three product contexts
3. Claude scores each product against your requirements from each product's knowledge store
4. Output: scored gap matrix per vendor, cited to product documentation

---

## Repository structure

```
composable-ai-skills/
│
├── framework/                        ← domain-agnostic core — start here
│   ├── README.md                     ← framework concepts and design
│   ├── SKILL_BUILDING_GUIDE.md       ← complete how-to for skill builders
│   ├── CONTEXT_PROTOCOL.md           ← shared context specification (normative)
│   ├── GUARDRAIL_FRAMEWORK.md        ← configurable compliance rules
│   ├── templates/
│   │   ├── PRODUCT_SKILL_TEMPLATE.md
│   │   └── CAPABILITY_SKILL_TEMPLATE.md
│   └── examples/
│       └── minimal-example/
│
├── implementations/                  ← domain-specific skill libraries
│   └── gxp-lab-automation/           ← reference implementation (in development)
│       ├── README.md
│       ├── product-skills/           ← one folder per product
│       ├── capability-skills/        ← one folder per capability
│       └── guardrail-profiles/       ← domain compliance configuration
│
├── CONTRIBUTING.md                   ← how to contribute + legal and IP guidance
└── LICENSE                           ← MIT
```

---

## Framework documents

| Document | Purpose | Audience |
|---|---|---|
| [SKILL_BUILDING_GUIDE.md](framework/SKILL_BUILDING_GUIDE.md) | Complete how-to for building a skill from scratch | Domain experts + developers |
| [CONTEXT_PROTOCOL.md](framework/CONTEXT_PROTOCOL.md) | Exact specification for context blocks and handoff signals | Skill builders |
| [GUARDRAIL_FRAMEWORK.md](framework/GUARDRAIL_FRAMEWORK.md) | Seven configurable compliance guardrail slots | Skill builders |
| [PRODUCT_SKILL_TEMPLATE.md](framework/templates/PRODUCT_SKILL_TEMPLATE.md) | Fill-in-the-blanks starting point for product skills | Skill builders |
| [CAPABILITY_SKILL_TEMPLATE.md](framework/templates/CAPABILITY_SKILL_TEMPLATE.md) | Fill-in-the-blanks starting point for capability skills | Skill builders |

---

## The reference implementation

`implementations/gxp-lab-automation/` is a complete skill library for
lab automation systems — LIMS, CDS, SDMS, ELN — in GxP-regulated
pharmaceutical environments. It demonstrates every framework concept
applied to a real, complex, regulated domain.

### Planned skills

#### Product skills
| Skill | System | Versions | Status |
|---|---|---|---|
| `waters-empower` | Waters Empower CDS | 2 · 3 FR3 · FR4 · FR5 | In development |
| `labware-lims` | LabWare LIMS | v6.x | Planned |
| `labvantage-lims` | LabVantage LIMS | 8.x | Planned |
| `starlims` | STARLIMS | v12 | Planned |
| `chromeleon-cds` | Thermo Chromeleon | 7.x | Planned |
| `benchling-eln` | Benchling ELN | Current | Planned |

#### Capability skills
| Skill | What it does | Status |
|---|---|---|
| `gxp-sdlc-coauthor` | URS · FRS · IQ/OQ/PQ · change control · test scripts · RTM | In development |
| `product-evaluation` | URS fit scoring · vendor gap matrix | Planned |
| `support-assist` | Incident triage · runbooks · root cause | Planned |
| `data-integrity-audit` | ALCOA+ review · audit trail checks | Planned |
| `inspection-readiness` | FDA/EMA prep · mock audit · response drafting | Planned |

---

## GxP guardrails — enforced by design

Every skill in the GxP lab automation implementation enforces seven
non-negotiable guardrails. These are not optional — they are reviewed
on every pull request:

| Guardrail | What it enforces |
|---|---|
| Source discipline | Claims grounded only in retrieved documentation — no general knowledge substitution |
| Fabrication firewall | Unverifiable content marked `[UNVERIFIED]`, never stated as fact |
| Version discipline | Documents retrieved only from the confirmed version folder |
| Conflict surfacing | Document vs KB contradictions shown to user — never silently resolved |
| Domain safety filter | Suggestions that bypass compliance controls flagged as unsafe |
| Freshness signalling | KB articles older than threshold flagged with retrieval date |
| Human decision gate | No autonomous skill chaining — user confirms every transition |

---

## Knowledge sources — your environment, your documents

Skills retrieve from two sources you configure:

**Document store** (SharePoint, shared drive, or equivalent) — your own
copies of vendor documentation organised by product and version. The skill
reads from paths you configure in the skill's config zone. No vendor
documentation is included in this repository.

**Vendor KB** (live web fetch) — vendor support portal URLs you register
in the skill. Fetched at query time. Every article is date-stamped.

You supply the documents from your own licensed sources. The framework
supplies the retrieval logic, citation rules, and compliance guardrails.

---

## Who builds what

**Framework contributors** — improve the core templates, protocol spec,
guardrail framework, and building guide. Changes here affect every skill
built on the framework and require careful review.

**Implementation contributors** — build skills for a specific domain using
the framework. A validation engineer who knows Empower deeply can build
a better Empower skill than a developer who does not.

**End users** — load skills into Claude, fill in the config zone with their
own paths and URLs, and use the resulting composed expert assistant.

---

## Where to start

**Use existing skills** → go to `implementations/` → find your domain →
read the implementation README → follow setup instructions.

**Build a product skill** → read [`SKILL_BUILDING_GUIDE.md`](framework/SKILL_BUILDING_GUIDE.md)
→ start from [`PRODUCT_SKILL_TEMPLATE.md`](framework/templates/PRODUCT_SKILL_TEMPLATE.md).

**Build a capability skill** → read [`SKILL_BUILDING_GUIDE.md`](framework/SKILL_BUILDING_GUIDE.md)
and [`CONTEXT_PROTOCOL.md`](framework/CONTEXT_PROTOCOL.md)
→ start from [`CAPABILITY_SKILL_TEMPLATE.md`](framework/templates/CAPABILITY_SKILL_TEMPLATE.md).

**Start a new domain implementation** → read all four framework documents
→ create a folder under `implementations/[your-domain]/` → start with one
product skill and one capability skill → validate the handoff before building more.

**Contribute or report an issue** → read [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the full authoring guide,
pull request process, and legal and IP guidance — including what to check
before using vendor documentation with these skills.

The fastest way to contribute is a product skill for a system you know well.
Use the template, follow the building guide, open a PR.

---

## Legal

MIT licence. See [`LICENSE`](LICENSE).

Product names used in skill files identify the systems those skills support.
This project is independent and community-maintained. It is not affiliated
with, endorsed by, or sponsored by any vendor whose products are referenced
in implementation skill files.

Users are responsible for ensuring their use of vendor documentation with
these skills is consistent with their licence agreements. This repository
contains no vendor documentation. See [`CONTRIBUTING.md`](CONTRIBUTING.md)
for full legal and IP guidance.

---

## Acknowledgements

Built on the idea that domain expertise — not engineering skill — is what
makes an AI assistant genuinely useful in a regulated environment. Every
skill contributed by someone who knows a system deeply makes Claude more
useful for everyone who works with that system.
