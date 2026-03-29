# [CAPABILITY NAME] — Capability Skill

<!--
INSTRUCTIONS FOR SKILL BUILDERS
================================
Replace every [PLACEHOLDER] with your content.
Delete all instruction blocks (marked like this one) before submitting.
Do not change section headings, section order, or delimiter formats.
Read SKILL_BUILDING_GUIDE.md and CONTEXT_PROTOCOL.md completely
before filling in this template.
-->

## Identity

**Skill type:** Capability skill — Tier 2  
**Function:** [One sentence: what this skill does — e.g. "Co-authors GxP SDLC documents grounded in the active product skill's knowledge store."]  
**Domain:** [Domain name — must match the domain of compatible product skills]  
**Guardrail profile:** [Profile name — must match compatible product skills]  
**Archetypes served:** [List — e.g. Validation Engineer · IT Admin]  
**Depends on:** A product skill must be loaded and its `[PRODUCT_CONTEXT]` block present in session. This skill cannot operate without product context.

**Compatible product skills:**
<!--
List every product skill this capability skill can work with.
Use exact skill_id values (folder names).
If compatible with all product skills in the domain, write "All [domain] product skills."
-->
- `[product-skill-id]` — [product name]
- `[product-skill-id]` — [product name]

**Context fields read:**
<!--
List every field this skill reads from the product context block.
This is a compatibility declaration — reviewers use it to verify
that product skills provide what this capability skill expects.
-->
This skill reads the following fields from `[PRODUCT_CONTEXT]`:
- `product`, `version` — used for document headers and citations
- `knowledge_path` — used for document retrieval
- `kb_url`, `kb_search` — used for knowledge base retrieval
- `archetype` — used to set response tone
- `domain`, `guardrail_profile` — used to apply domain guardrails
- [add any additional fields this skill reads]

**Trigger phrases:**
<!--
Phrases a user might say that should activate this skill.
Focus on the task, not the product — the product skill handles product triggers.
-->
```
"[task phrase]", "[document type]", "[action phrase]",
"[help me verb]", "[create/write/draft] [output type]"
```

---

## Disclaimer

DISCLAIMER: This skill is an independent, community-maintained resource.
It is not affiliated with, endorsed by, or sponsored by any vendor.
It is a general-purpose capability skill that operates on product context
provided by a product skill loaded by the user.

---

## Step 0 — Product context check

<!--
This is the first step. It is mandatory and must not be moved.
Do not modify the hard stop logic. You may adjust message text.
-->

Before taking any other action, check for a `[PRODUCT_CONTEXT]` block
in the session.

**If a context block is present:**
Read and store all required fields silently.
Confirm with a single line:
> "Working with: {product} {version} — {archetype} mode. Ready to [capability description]."

Do not ask the user to repeat any information already in the context block.

**If no context block is present:**
Stop completely. Write:
> "No product context found in this session.
> To use this skill, first load a product skill for your system.
> The product skill provides the version, document paths, and domain settings
> this capability skill requires to operate accurately."

Do not attempt to infer product context from the conversation.
Do not proceed without a formally declared `[PRODUCT_CONTEXT]` block.

**If multiple context blocks are present (comparative mode):**
<!--
Define how this skill handles multiple simultaneous product contexts.
Two options — choose one and delete the other:

Option A (ask user to choose one):
Ask the user which product to work with before proceeding.

Option B (work with all — for evaluation/comparison capability skills):
Read all context blocks. Work with all products simultaneously.
Attribute all retrieved information to the correct product.
-->
[Choose Option A or Option B and write the instruction here.]

---

## Step 1 — Mode selection

<!--
Define the task modes this capability skill supports.
Each mode is a distinct type of output this skill produces.
If your skill has only one mode, simplify this section accordingly.
-->

Ask the user which task they need, if not already stated:

> "Which [output type] do you need?
> 1. [Mode 1 name] — [one line description]
> 2. [Mode 2 name] — one line description]
> 3. [Mode 3 name] — [one line description]"

<!--
Add as many modes as your capability requires.
-->

Store selection as `[CAPABILITY_MODE]` in session context.

<!--
If the archetype affects how a mode behaves, note it here.
Example:
"In [Mode 1], archetype affects the level of technical detail:
- [Archetype 1] → include regulatory citations and document section references
- [Archetype 2] → focus on procedural steps and system commands"
-->

---

## Step 2 — Document retrieval

<!--
Define which documents to retrieve for each mode.
This step uses knowledge_path and kb_url from the product context block.
-->

Before beginning each mode, retrieve supporting documentation.

Navigate to `{knowledge_path}` from the product context block.

### Document lookup by mode

| Mode | Documents to retrieve |
|---|---|
| [Mode 1] | [document types] |
| [Mode 2] | [document types] |
| [Mode 3] | [document types] |

File handling:
- `.pdf` → pdf-reading skill
- `.docx` → file-reading skill (docx mode)
- `.xlsx` → file-reading skill (xlsx mode)
- `.txt` → read directly

Also fetch relevant KB articles from `{kb_search}` using terms from the
user's query. Record article URL, title, retrieval date, and last updated date.

If a required document is not found:
> "The expected document [document name] was not found in your configured knowledge store.
> I will note this as a gap rather than proceed without the source."

---

## Step 3 — Mode execution

<!--
Define each mode in full. Each mode is a subsection.
Include:
- Opening prompt to the user
- Structure of the output produced
- How the output is built (iteratively / populated from documents / delta analysis / assembled)
- Citation requirement for this mode
- How this mode knows it is complete

The GxP SDLC co-author reference implementation shows what detailed
mode execution looks like. Use it as a model for the level of specificity required.
-->

---

### MODE: [Mode 1 name]

**Approach:** [One sentence: how this mode builds its output — e.g. iterative elicitation, document-population, delta analysis, assembly from session artifacts]

**Opening prompt:**
> "[Question or instruction to start this mode]"

**Output structure:**

<!--
Define the structure of the document or artefact this mode produces.
Use numbered sections. This structure is what Claude builds, section by section.
-->

```
[Output title — {product} {version}]

1. [Section name]
   1.1 [Subsection]
   1.2 [Subsection]

2. [Section name]
   ...
```

**How the output is built:**

<!--
Describe the authoring approach precisely.
Examples:
- "Claude asks the user a question for each section, builds the content from the answer"
- "Claude populates each section from the retrieved [document type], citing the source"
- "Claude compares current state (from context) against proposed change (from user) and populates the impact assessment"
-->

[Describe the authoring approach.]

**Citation requirement:**
Every [claim / step / expected result / requirement / assessment] in this output
must be cited using the standard citation format:
`[Source: {filename}, {version path}, §{section} or p.{page}]`
`[KB: {article URL or ID}, retrieved {date}, last updated {date}]`

Unverifiable content: `[UNVERIFIED — no source found in configured knowledge store]`

---

### MODE: [Mode 2 name]

<!--
Repeat the mode structure for each additional mode.
-->

**Approach:** [One sentence]

**Opening prompt:**
> "[Opening question or instruction]"

**Output structure:**

```
[Output title]

1. [Section name]
...
```

**How the output is built:**

[Describe the authoring approach.]

**Citation requirement:**
[Define citation requirement for this mode.]

---

<!--
Add MODE sections for all remaining modes.
-->

---

## Step 4 — Section confirmation loop

<!--
This step is mandatory for all capability skills that produce structured documents.
It is the mechanism that keeps the human in the loop during authoring.
Do not weaken or remove it.
-->

After completing each section of the output, pause and ask:

> "Section {n} — {section title} is drafted above.
> Please review and confirm, or tell me what to change.
> Type 'confirmed' to move to Section {n+1}, or describe your amendments."

Do not advance to the next section without explicit user confirmation.
Do not skip sections to improve speed.
Do not offer to generate the complete document in one pass.

The confirmation loop exists so the user reviews every section before the
output is considered complete. This is a governance requirement, not a UX choice.

---

## Step 5 — Output summary and handoff

When all sections are confirmed, write the output summary block:

```
[CAPABILITY_OUTPUT]
skill:           [your-skill-folder-name]
product:         {product from product context}
version:         {version from product context}
mode:            {CAPABILITY_MODE}
output_title:    {title of the produced document or report}
req_ids:         {comma-separated list of requirement IDs generated, or NONE}
tc_ids:          {comma-separated list of test case IDs generated, or NONE}
sources_used:    {list of document filenames and KB article IDs cited}
gaps_flagged:    {count of [UNVERIFIED] markers — list them if count > 0}
status:          Draft — pending review
[/CAPABILITY_OUTPUT]
```

Then write the handoff signal:

```
[CAPABILITY_HANDOFF]
from_skill:      [your-skill-folder-name]
output:          {output_title}
gaps:            {count and list if > 0, else NONE}
suggested_next:  {skill_id of recommended next skill, or NONE}
reason:          {one sentence, or NONE}
user_action:     {instruction to user — what to do next}
[/CAPABILITY_HANDOFF]
```

**`status` is always "Draft — pending review"** until the user explicitly
states the document is ready for signature. Never mark output as final,
approved, or complete.

---

## Guardrails

<!--
Implement all seven guardrails per the GUARDRAIL_FRAMEWORK.md.
Copy your domain's guardrail profile configuration and add
capability-specific additions where needed.
Do not weaken any guardrail relative to the profile.
-->

**Guardrail profile applied:** [profile name]

1. **Source discipline**
   [Copy from guardrail profile.]
   [Add: specific sources this capability skill retrieves — document types, KB endpoints.]

2. **Fabrication firewall**
   Marker: `[UNVERIFIED — no source found in configured knowledge store]`
   [Add: the specific categories of claim most at risk in this capability —
   e.g. expected results in test cases, regulatory thresholds in requirements,
   impact assessments in change control.]

3. **Version discipline**
   [Copy from guardrail profile.]
   [Add: this capability skill reads version from context block — it does not
   confirm version independently. If the context block version is wrong,
   the user must reload the product skill with the correct version.]

4. **Conflict surfacing**
   [Copy from guardrail profile.]

5. **Domain safety filter**
   [Copy from guardrail profile.]
   [Add: any capability-specific safety considerations — e.g. in a document
   authoring capability, flag if a requirement would mandate disabling a control.]

6. **Freshness signalling**
   [Copy from guardrail profile.]

7. **Human decision gate**
   [Copy from guardrail profile.]
   [Add: section-by-section confirmation loop (Step 4) as an additional
   confirmation point within each document mode.]
   No autonomous skill chaining. Handoff signal always waits for user action.
   Output status always "Draft — pending review" until user confirms otherwise.
