# Skill building guide

This guide is for anyone building a skill using the Composable AI Skills
Framework — whether you are a domain expert who has never written a technical
specification, or a developer who wants to understand the design constraints.

Read this guide completely before opening a template. Understanding why the
framework is structured the way it is will make you a better skill builder.

---

## Part 1 — Concepts (read this first)

### What you are building

A skill is a single markdown file that tells Claude how to behave when that
skill is loaded into a session. It is not software. It does not execute.
It is a precise set of instructions written in plain English.

The quality of a skill depends almost entirely on the quality of its
instructions — how precisely they define what Claude should retrieve,
how it should reason about what it finds, what it should produce, and what
it must never do. Domain expertise matters more than technical skill here.

### The two skill types

**Product skill (Tier 1)** — encodes deep knowledge of a specific system.
What it is, how it is structured, what its compliance-critical configuration
points are, where things go wrong, which version differences matter. This
skill is passive — it does not produce documents or triage incidents. It
establishes a rich context that capability skills consume.

**Capability skill (Tier 2)** — encodes how to do something. Evaluate
systems against requirements, co-author compliance documents, triage
support incidents, prepare for regulatory inspections. This skill is active
— it retrieves information, synthesises it, and produces structured output.
It cannot operate without a product skill providing context.

### Why two types instead of one

Separation of concerns. A validation engineer building a product skill for
a LIMS system should not need to understand how a document co-authoring
capability works. A developer improving the co-authoring capability should
not need to know LIMS internals.

More importantly: composability. If product knowledge and task capability
were combined in one skill, you would need a separate skill for every
combination of product and task — LabWare LIMS + evaluation, LabWare LIMS
+ co-author, LabWare LIMS + support, and so on, multiplied across every
product. With two tiers, adding a new product skill makes it immediately
work with every existing capability skill. Adding a new capability skill
makes it immediately work with every existing product skill.

### The config zone

Every skill has a config zone — a small block at the top of the fixed
section that contains placeholder variables for environment-specific
settings: document store paths, knowledge base URLs, system identifiers.

The user fills in these placeholders for their environment. The skill
logic references the variables by name. This means the same skill works
in any environment without modifying the logic.

The config zone is the only part of a skill that the end user edits.
The fixed zone is never modified by the user.

### How skills communicate

Skills communicate through the session context — the conversation that
Claude and the user share. There is no API between skills. There is no
function call. There is no message bus.

When a product skill loads, it writes a structured context block into the
conversation. When a capability skill loads, it reads that context block.
The context block is the handshake. Its format is defined precisely in
`CONTEXT_PROTOCOL.md` and must be followed exactly.

When a skill completes a phase, it writes a handoff signal — a structured
block that summarises what was produced and suggests what comes next.
The user reads the handoff signal and decides whether to load the suggested
next skill. Skills never make that decision autonomously.

---

## Part 2 — Before you write anything

### Check what already exists

Before building a new skill, verify:

1. Is there already a skill for this product or capability in the repository?
   If yes, consider contributing to it rather than creating a duplicate.

2. Is there a skill for a similar product that you could model yours on?
   For a new CDS system, study the existing CDS product skills. The structure
   will be similar. The product-specific knowledge will differ.

3. Which capability skills will your product skill need to work with?
   Read those capability skills. Understand what context fields they expect.
   Make sure your product skill's context block provides all of them.

### Define your skill's scope

Before writing, answer these questions:

**For a product skill:**
- What is the exact product name and vendor?
- Which versions will this skill cover? What are the meaningful differences between versions?
- Who will use this skill? (Domain expert, IT admin, evaluator, auditor — or multiple)
- What are the compliance-critical aspects of this product in your target domain?
- What are the most common mistakes, misconfigurations, or inspection findings?
- Where does authoritative documentation live? What types of documents exist?
- Which knowledge base URLs are relevant?
- Which capability skills is this product skill most likely to be paired with?

**For a capability skill:**
- What task does this skill perform?
- What product context does it need to operate? (List every context field it will read)
- What does it produce? (Documents, reports, analyses — describe the outputs precisely)
- Which archetypes does it serve? Does it behave differently for different archetypes?
- What are the compliance constraints on this task in the target domain?
- How does it know when it has completed a phase and should hand off?

Write the answers down before opening the template. Skills built without this
clarity are invariably incomplete or inconsistent.

### Understand your domain's guardrails

The Guardrail Framework (`GUARDRAIL_FRAMEWORK.md`) defines seven guardrail
slots that every skill must implement. Each slot has a domain-specific
configuration. Before writing your skill, read the Guardrail Framework and
decide what each slot means for your domain.

For a GxP lab automation domain the fabrication guardrail means: if a step
cannot be sourced from the retrieved documentation, mark it as unverified
and do not state it as fact.

For a financial compliance domain the same guardrail slot means: if a
calculation or regulatory threshold cannot be sourced from a retrieved
regulation or standard, mark it as unverified.

The mechanism is the same. The domain content differs. Decide your
domain's guardrail content before writing the skill.

---

## Part 3 — Building a product skill

### Step 1 — Open the template

Copy `framework/templates/PRODUCT_SKILL_TEMPLATE.md` to your skill folder.
Do not start from scratch. The template contains mandatory sections in the
mandatory order. Deviating from the section order breaks compatibility with
capability skills that read the context block.

### Step 2 — Fill in the Identity section

The identity section tells Claude and other builders what this skill is.
Fill in every field. Do not leave placeholders.

Required fields:
- Skill type (always "Product skill — Tier 1")
- Product name (exact, as the vendor uses it)
- Vendor name
- Versions covered (list each version that has meaningfully different behaviour)
- Domain (the regulated or professional domain this skill operates in)
- Archetypes served (who will use this skill)
- Companion capability skills (which capability skills this product skill is designed to work with)
- Trigger phrases (the phrases a user might say that should activate this skill)

**On trigger phrases:** These are important. They are how Claude recognises
that the user's query is relevant to this skill. Include the full product
name, common abbreviations, version identifiers, and the kinds of questions
users actually ask. Do not be too narrow.

Example — good trigger phrases:
```
"Empower", "Empower 3", "Waters CDS", "chromatography data system",
"Empower audit trail", "Empower upgrade", "FR5", "Empower node",
"Empower 21 CFR Part 11", "Empower OQ"
```

Example — too narrow:
```
"Waters Empower 3 FR5 chromatography data system"
```

### Step 3 — Write the disclaimer

Copy the standard disclaimer verbatim from the template. Do not modify it.
The disclaimer must appear in every skill file immediately after the
identity section.

### Step 4 — Fill in the config zone

Define the placeholder variables that the end user will fill in. Every
variable must follow these rules:

- UPPERCASE_WITH_UNDERSCORES naming
- One variable per line
- Each variable has a plain English description in square brackets
- Never populate with real values — placeholders only
- Never include credentials, tokens, or authentication details

Mandatory config variables for all product skills:
```
KNOWLEDGE_PATH:    [path to your document store root folder]
KB_URL:            [your vendor knowledge base URL]
KB_SEARCH:         [your vendor KB search endpoint URL, if available]
```

Add version-specific path variables for each version your skill covers:
```
PATH_V[version]:   [path to version-specific subfolder, or BLANK if not used]
```

Add any domain-specific variables your retrieval protocol requires.

The config zone must end with the exact delimiter:
```
## ── END CONFIGURATION — DO NOT EDIT BELOW THIS LINE ──
```

### Step 5 — Write the configuration check (Step 0)

This is the first thing Claude does when the skill is invoked. It checks
that the config zone has been filled in before attempting any retrieval.

Required behaviour:
- If all paths and URLs are blank → ask the user for them before proceeding
- If some are blank → proceed with what is configured, note what is missing
- If KB_URL is blank → proceed using document store only, note KB is unavailable
- Never proceed with retrieval against an empty KNOWLEDGE_PATH
- Never fabricate paths or URLs

### Step 6 — Write the version confirmation (Step 1)

For product skills, version confirmation is mandatory before any document
retrieval. The skill must ask the user which version they are running and
map the answer to the correct PATH_V[version] variable.

The confirmed version must be stored in the context block as `version`.
The confirmed path must be stored as `knowledge_path`.

Include a warning:
```
Never retrieve documents from a version path that does not match the
confirmed version. Cross-version retrieval is a framework violation.
```

### Step 7 — Write the context declaration (Step 2)

This is the most critical section of any product skill. The context
declaration writes the structured context block that all capability
skills depend on. Its format must exactly match the specification in
`CONTEXT_PROTOCOL.md`.

The context block must be written visibly in the conversation —
not silently. The user must be able to see and confirm it.

Required fields — do not add, remove, or rename these:
```
[PRODUCT_CONTEXT]
framework_version:  1.0
skill_id:           [your-skill-folder-name]
product:            [exact product name]
vendor:             [exact vendor name]
version:            [confirmed by user]
knowledge_path:     [resolved from config]
kb_url:             [from config]
kb_search:          [from config]
domain:             [your domain name]
guardrail_profile:  [your domain guardrail profile name]
archetype:          [confirmed or asked]
session_date:       [today's date]
[/PRODUCT_CONTEXT]
```

You may add domain-specific fields after `session_date`. Do not insert
fields before it or between mandatory fields.

### Step 8 — Write the retrieval protocol (Steps 3 and 4)

The retrieval protocol tells Claude exactly how to retrieve information
when it needs to answer a query or populate a document. It has two parts:

**Document store retrieval:**

Define a lookup table: for each type of query or task the user might bring,
which documents in the knowledge store should be retrieved? Be specific.
Name the document types, not just "the documentation."

Define file handling rules:
- Which file extensions map to which reading approach
- In what order to read documents if multiple are relevant
- What to do if a document is not found in the configured path

**Knowledge base retrieval:**

Define how Claude constructs search queries from user questions.
Define what to record for every article retrieved (URL, title, date retrieved,
date last updated).
Define the stale content threshold for your domain.

### Step 9 — Write the synthesis rules (Step 4)

Synthesis rules tell Claude how to compose its response from retrieved content.

Every product skill must include:

**Fabrication firewall** — the exact wording Claude uses when it cannot
ground a claim in retrieved content. This is mandatory and must use the
`[UNVERIFIED]` marker defined in the Guardrail Framework.

**Domain credibility filter** — which types of retrieved content require
a domain safety warning (e.g. workarounds that bypass compliance controls,
community posts that contradict official documentation).

**Archetype tone** — how Claude's response style changes for each
archetype the skill serves.

### Step 10 — Write the citation format (Step 5)

Every product skill must define the citation format its responses use.
The format must follow the framework standard with allowed domain extensions:

Document store citation:
```
[Source: {filename}, {version path}, §{section} or p.{page}]
```

Knowledge base citation:
```
[KB: {article ID or URL}, retrieved {date}, last updated {date}]
```

Unverified content marker:
```
[UNVERIFIED — no source found in configured knowledge store]
```

Do not invent alternative citation formats. The standard format is
what capability skills and end users expect to see.

### Step 11 — Write the product knowledge section

This is where domain expertise matters most. Write the background knowledge
Claude needs to contextualise retrieved documents and answer questions that
do not require retrieval.

Organise this section by topic, not by document. Good topics for a product skill:

- Version landscape and what is significant about each version
- System architecture (components, their roles, how they connect)
- Data hierarchy or information model
- Compliance-critical configuration points for your domain
- Common failure modes, misconfigurations, inspection findings
- Integration touch points with other systems
- Common upgrade patterns and their risks

**Important:** Label this section clearly as background knowledge, not as
citable fact. Every specific claim a user might act on must come from
retrieved documentation, not from this section. This section gives Claude
the understanding to retrieve correctly and reason about what it finds.

### Step 12 — Write the handoff signal format

Define the exact handoff signal block this skill writes when the user's
request moves into capability skill territory.

The handoff signal must follow the framework standard:
```
[PRODUCT_HANDOFF]
from_skill:      [this skill's skill_id]
context_passed:  [summary of what is in the context block]
output_summary:  [what was established or produced in this session]
suggested_next:  [skill_id of the recommended next skill]
reason:          [why that skill is the right next step]
user_action:     Load the suggested skill to continue. Your context will carry over.
[/PRODUCT_HANDOFF]
```

Write example handoff signals for the two or three most common transitions
from this product skill to a capability skill. Examples make the intent
clear to users.

### Step 13 — Write the guardrail declarations

Copy the seven guardrail slots from the Guardrail Framework and fill in
the domain-specific content for each. This is mandatory. The seven slots
are not optional. Do not weaken, combine, or remove any of them.

The guardrail section must be the last section of the fixed zone.

---

## Part 4 — Building a capability skill

Most of the guidance above applies to capability skills as well.
The differences are:

### No version confirmation

Capability skills do not confirm product versions — that is the product
skill's responsibility. The capability skill reads the confirmed version
from the context block.

### Context reading instead of context writing

The first thing a capability skill does is read the product context block.
If no context block is present, the skill must stop and ask the user to
load a product skill first.

The capability skill must explicitly name every context field it reads.
This is how skill builders and reviewers can verify compatibility between
product and capability skills.

### Multiple document modes

Most capability skills serve multiple task types — writing different
kinds of documents, performing different kinds of analysis. Each task
type is a separate document mode. Define each mode explicitly:

- Mode name and trigger
- What the mode produces
- What it reads from the context block
- What documents it retrieves from the knowledge store
- How it structures its output
- How it knows it has completed a phase

### Confirmation loop

Capability skills that produce structured documents must include a
section-by-section confirmation loop. Claude produces one section,
asks the user to confirm, waits for confirmation, then proceeds to
the next section.

This is mandatory for document-producing capability skills. It is the
mechanism that keeps the human in the loop during document authoring.

### Output summary block

When a capability skill completes its task, it writes an output summary
block before the handoff signal:

```
[CAPABILITY_OUTPUT]
skill:           [this skill's skill_id]
product:         [from product context]
version:         [from product context]
mode:            [document mode used]
output_title:    [title of the produced document or report]
sources_used:    [list of documents and KB articles cited]
gaps_flagged:    [number of UNVERIFIED markers — list them if any]
status:          Draft — pending review
[/CAPABILITY_OUTPUT]
```

Then write the handoff signal using the CAPABILITY_HANDOFF format
defined in `CONTEXT_PROTOCOL.md`.

---

## Part 5 — Quality checklist

Before submitting a skill for review, verify every item:

### Structure
- [ ] Starts from the correct template
- [ ] All mandatory sections present in the correct order
- [ ] Config zone has correct delimiter format
- [ ] Config zone contains only empty placeholders — no real values
- [ ] Standard disclaimer present verbatim

### Context protocol
- [ ] Context block uses exact field names from CONTEXT_PROTOCOL.md
- [ ] No mandatory fields added, removed, or renamed
- [ ] Context block is written visibly, not silently (product skills)
- [ ] Context block is read before any action (capability skills)
- [ ] Handoff signal uses the correct format for skill type

### Retrieval
- [ ] Document store retrieval covers all major query types
- [ ] File type routing defined for all file types in the knowledge store
- [ ] KB retrieval records article URL, title, retrieval date, last updated date
- [ ] Stale content threshold defined and consistent with domain guardrail profile

### Guardrails
- [ ] All seven guardrail slots implemented
- [ ] Fabrication firewall uses [UNVERIFIED] marker
- [ ] Domain credibility filter defined
- [ ] No guardrail is weakened relative to the Guardrail Framework default

### Citations
- [ ] Citation format matches framework standard
- [ ] Every synthesis rule references citation requirement
- [ ] Unverified content marker is [UNVERIFIED] exactly

### Handoff
- [ ] Handoff signal format matches CONTEXT_PROTOCOL.md
- [ ] Suggested_next references a real skill_id
- [ ] User_action field always instructs user to make the next decision
- [ ] No autonomous chaining — skill never loads the next skill itself

### Product skill specific
- [ ] Version confirmation step present and mandatory
- [ ] Version-to-path mapping covers all declared versions
- [ ] Cross-version retrieval warning present
- [ ] Product knowledge section labelled as background, not citable fact

### Capability skill specific
- [ ] Context check is Step 0 — first thing the skill does
- [ ] Hard stop if no product context found
- [ ] Every context field the skill reads is named explicitly
- [ ] Confirmation loop present for document-producing modes
- [ ] Output summary block written before handoff signal

### CHANGELOG.md
- [ ] CHANGELOG.md present in skill folder
- [ ] Initial entry for version 1.0.0
- [ ] Semver versioning rules followed

---

## Part 6 — Common mistakes

**Combining product knowledge and capability logic in one skill.**
This destroys composability. Keep them separate. If you find yourself
writing retrieval protocol AND document authoring logic in the same file,
split them.

**Writing trigger phrases that are too specific.**
Users do not say "Waters Empower 3 FR5 chromatography data system audit trail."
They say "Empower audit trail" or "why does my Empower OQ keep failing."
Write trigger phrases that match how practitioners actually speak.

**Populating the config zone with real paths or URLs.**
These must stay in the user's environment. If you commit real paths,
you have broken the separation between framework and environment.
You have also potentially exposed internal infrastructure details.

**Writing product knowledge as citable fact.**
The product knowledge section gives Claude background understanding.
It is not a source. Specific claims — configuration steps, error codes,
compliance thresholds — must come from retrieved documents. Label the
product knowledge section clearly so neither Claude nor the user treats
it as authoritative.

**Weakening the fabrication guardrail.**
The most common guardrail mistake is writing synthesis rules that say
"if you cannot find the answer in retrieved documents, use your general
knowledge." This breaks the entire grounding model. If the answer is
not in a retrieved source, it must be marked [UNVERIFIED].

**Creating your own citation format.**
The framework citation format exists so that all skills produce citations
in the same structure. Users learn to read one format. Capability skills
know where to find source references. Do not invent variants.

**Writing a handoff signal that loads the next skill autonomously.**
The handoff signal suggests. The user decides. Write the user_action
field as an instruction to the user, never as an instruction to Claude.

**Skipping the confirmation loop in a document-producing capability skill.**
In any domain where documents have consequences — legal, regulatory,
operational — the user must confirm each section before Claude advances.
The confirmation loop is not a UX nicety. It is a governance requirement.

---

## Part 7 — Getting help

If you are stuck on a section of your skill, the reference implementation
in `implementations/gxp-lab-automation/` shows every concept applied in
a real, complex domain. Study the waters-empower product skill and the
gxp-sdlc-coauthor capability skill. The patterns they implement are the
patterns the framework requires.

If the reference implementation does not cover your question, open a
discussion in the repository. Describe the domain you are building for,
the specific section you are working on, and what you have tried.

If you believe the framework itself needs to change to accommodate a valid
use case, open an issue labelled "framework proposal." Framework changes
require maintainer review because they affect every skill built on it.
