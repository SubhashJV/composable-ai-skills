# Context protocol specification

Version: 1.0  
Status: Normative — all skills built on this framework must implement this specification exactly.

This document defines the structured context blocks and handoff signals that
skills use to communicate through the shared session context. Deviating from
this specification breaks compatibility between skills.

---

## Overview

Skills communicate through the conversation that Claude and the user share.
There is no direct call between skills. Communication happens through
structured text blocks written into and read from the session context.

Three block types are defined:

| Block | Written by | Read by | Purpose |
|---|---|---|---|
| `[PRODUCT_CONTEXT]` | Product skill | Capability skills | Passes product, version, paths, domain settings |
| `[PRODUCT_HANDOFF]` | Product skill | User | Signals phase complete, suggests next skill |
| `[CAPABILITY_OUTPUT]` | Capability skill | User + next capability skill | Summarises what was produced |
| `[CAPABILITY_HANDOFF]` | Capability skill | User | Signals phase complete, suggests next skill |

---

## 1. Product context block

### When to write

The product skill writes this block immediately after version confirmation
(Step 2 of the product skill protocol). It must be written visibly in the
conversation — the user must be able to read and confirm it.

### Format — exact

```
[PRODUCT_CONTEXT]
framework_version:  1.0
skill_id:           {skill folder name — e.g. waters-empower}
product:            {exact product name as vendor uses it}
vendor:             {vendor / organisation name}
version:            {version confirmed by user}
knowledge_path:     {resolved path from config — version-specific}
kb_url:             {KB_URL from config, or NONE if not configured}
kb_search:          {KB_SEARCH from config, or NONE if not configured}
domain:             {domain name — e.g. gxp-lab-automation}
guardrail_profile:  {guardrail profile name — e.g. gxp-pharma}
archetype:          {validation_engineer | it_admin | evaluator | other}
session_date:       {today's date in ISO 8601 — YYYY-MM-DD}
[/PRODUCT_CONTEXT]
```

### Rules

**Mandatory fields** — all fields above are mandatory. If a value is not
available (e.g. kb_url not configured), write `NONE`. Never omit a field.

**Field names** — field names are case-sensitive and must match exactly.
Do not abbreviate, expand, or rename them.

**Ordering** — fields must appear in the order listed. Do not reorder.

**Extension fields** — domain-specific fields may be added after
`session_date`. They must not be inserted between mandatory fields.
Extension field names must be lowercase with underscores.
Example extension for GxP domain:
```
gxp_regulation:     21_CFR_Part_11
gxp_gamp_category:  4
```

**Multiple product skills** — when more than one product skill is loaded
simultaneously (e.g. for comparative evaluation), each writes its own
context block with a distinct `skill_id`. Capability skills read all
present context blocks and attribute information to the correct product.

```
[PRODUCT_CONTEXT]
skill_id:     labware-lims
product:      LabWare LIMS
...
[/PRODUCT_CONTEXT]

[PRODUCT_CONTEXT]
skill_id:     labvantage-lims
product:      LabVantage LIMS
...
[/PRODUCT_CONTEXT]
```

### What capability skills do with the context block

A capability skill reads the context block at Step 0 — before taking any
other action. It must:

1. Verify a context block is present. If absent, stop and ask the user
   to load a product skill first.
2. Read and store all mandatory field values silently.
3. Confirm to the user in a single line what it has read:
   `"Working with: {product} {version} — {archetype} mode. Ready."`
4. Never ask the user to repeat information already in the context block.

---

## 2. Product handoff signal

### When to write

The product skill writes a handoff signal when:
- The user's request moves beyond product knowledge into task execution
  (authoring, evaluation, triage, audit)
- The product skill has completed establishing context and the next step
  requires a capability skill

The handoff signal is not written after every response. It is written
when a meaningful phase boundary is reached.

### Format — exact

```
[PRODUCT_HANDOFF]
from_skill:      {this skill's skill_id}
context_passed:  {one line: what is in the context block}
output_summary:  {one line: what was established or produced}
suggested_next:  {skill_id of the recommended next skill}
reason:          {one sentence: why that skill is the right next step}
user_action:     Load the {suggested_next} skill to continue. Your context will carry over.
[/PRODUCT_HANDOFF]
```

### Rules

**`user_action` must instruct the user, not Claude.** The skill never
loads the next skill autonomously. The user_action field is always
an instruction to a human.

**`suggested_next` must reference a real skill_id** — the exact folder
name of a skill that exists or is declared as planned in the repository.
Do not suggest a skill by description ("a document authoring skill").

**One handoff signal per phase.** Do not write multiple handoff signals
for different possible next steps. Choose the most appropriate one.
If multiple paths are genuinely equally valid, describe them in prose
before the handoff signal, then write the signal for the primary path.

---

## 3. Capability output block

### When to write

The capability skill writes an output block when it has completed a
document, report, analysis, or structured artefact. This block is written
before the handoff signal.

### Format — exact

```
[CAPABILITY_OUTPUT]
skill:           {this skill's skill_id}
product:         {product field from product context block}
version:         {version field from product context block}
mode:            {document mode name — e.g. oq_protocol}
output_title:    {title of the produced document or report}
req_ids:         {comma-separated list of requirement IDs, or NONE}
tc_ids:          {comma-separated list of test case IDs, or NONE}
sources_used:    {list of document filenames and KB article IDs cited}
gaps_flagged:    {integer count of [UNVERIFIED] markers in the output}
status:          Draft — pending review
[/CAPABILITY_OUTPUT]
```

### Rules

**`gaps_flagged` must be accurate.** Count every `[UNVERIFIED]` marker
in the produced output. If the count is greater than zero, list the
specific gaps in the output block so the user knows what to resolve.

**`status` is always "Draft — pending review"** until the user explicitly
states the document is ready for signature. The capability skill never
marks a document as approved, final, or complete.

**`req_ids` and `tc_ids`** are used by subsequent capability skills
(e.g. an RTM generator) to build traceability matrices. They must be
accurate lists of the IDs generated in this session.

---

## 4. Capability handoff signal

### When to write

Immediately after the capability output block. Follows the same rules
as the product handoff signal.

### Format — exact

```
[CAPABILITY_HANDOFF]
from_skill:      {this skill's skill_id}
output:          {output_title from capability output block}
gaps:            {gap count and brief list if count > 0, else NONE}
suggested_next:  {skill_id of the recommended next skill, or NONE}
reason:          {one sentence, or NONE if no next skill suggested}
user_action:     {instruction to user — what to do next}
[/CAPABILITY_HANDOFF]
```

**`suggested_next: NONE`** is valid. Not every capability output leads
to another skill. If the task is complete, say so.

---

## 5. Context reading rules for capability skills

These rules are mandatory. Every capability skill must implement them.

### Rule 1 — Context first

Reading the product context block is Step 0 of every capability skill.
Nothing else happens before it.

### Rule 2 — Hard stop if absent

If no `[PRODUCT_CONTEXT]` block is present in the session, the capability
skill must stop completely and write:

```
No product context found in this session.
To use this skill, first load a product skill for your system.
A product skill provides the version, document paths, and domain
settings this capability skill requires to operate accurately.
```

Do not attempt to infer product context from the conversation.
Do not proceed without a formally declared context block.

### Rule 3 — Never re-ask what context provides

If the product context block contains a field value, the capability skill
must not ask the user for that same information. The context block is
the authoritative source. Asking again implies the context was not read,
which erodes user trust in the skill system.

### Rule 4 — Explicit field declaration

Every capability skill must declare in its documentation which context
fields it reads. This declaration is used by skill builders and reviewers
to verify compatibility.

Example declaration format:
```
## Context fields read
This skill reads the following fields from the product context block:
- product, version — used for document headers and citations
- knowledge_path — used for document retrieval in Steps 2 and 3
- kb_url, kb_search — used for knowledge base retrieval in Step 3
- archetype — used to set response tone throughout
- domain, guardrail_profile — used to apply domain guardrails
```

### Rule 5 — Multi-product sessions

When multiple product context blocks are present, the capability skill
must attribute all retrieved information to the correct product.
It must not merge or conflate information from different products.

For evaluation capability skills, each product's information appears
in a named column or clearly attributed section. For co-authoring and
support capability skills, the skill must ask the user which product
to work with if more than one context block is present.

---

## 6. Session context lifecycle

Understanding the full lifecycle helps skill builders design handoffs correctly.

```
User loads product skill
        │
        ▼
Product skill: Step 0 — config check
        │
        ▼
Product skill: Step 1 — version confirmation
        │
        ▼
Product skill: Step 2 — writes [PRODUCT_CONTEXT] block  ◄── context available from here
        │
        ▼
Product skill: Steps 3–5 — retrieval, synthesis, citation
        │
        ▼ (when phase boundary reached)
Product skill: writes [PRODUCT_HANDOFF]
        │
        ▼ (user loads capability skill)
Capability skill: Step 0 — reads [PRODUCT_CONTEXT] block
        │
        ▼
Capability skill: Steps 1–N — mode selection, retrieval, authoring
        │
        ▼ (when output is complete)
Capability skill: writes [CAPABILITY_OUTPUT]
        │
        ▼
Capability skill: writes [CAPABILITY_HANDOFF]
        │
        ▼ (user decides next action)
```

The `[PRODUCT_CONTEXT]` block persists in the session for the entire
conversation. A capability skill loaded later in the session reads the
same context block written earlier. The context does not expire or refresh
unless the product skill is explicitly reloaded.

---

## 7. Versioning this specification

This specification is versioned. The `framework_version` field in every
context block records which version of the specification was used.

Current version: **1.0**

Changes to this specification that add mandatory fields or change field
names are breaking changes. They require a major version increment and
a migration guide for existing skills.

Changes that add optional extension fields or clarify existing behaviour
are non-breaking. They require a minor version increment.

Skills declaring `framework_version: 1.0` are compatible with all other
skills declaring `framework_version: 1.0`. A skill built against a higher
major version is not guaranteed to be compatible with 1.0 skills.
