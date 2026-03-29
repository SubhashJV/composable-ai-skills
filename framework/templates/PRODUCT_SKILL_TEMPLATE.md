# [PRODUCT NAME] — Product Skill

<!--
INSTRUCTIONS FOR SKILL BUILDERS
================================
Replace every [PLACEHOLDER] with your content.
Delete all instruction blocks (marked like this one) before submitting.
Do not change section headings, section order, or delimiter formats.
Read SKILL_BUILDING_GUIDE.md completely before filling in this template.
-->

## Identity

**Skill type:** Product skill — Tier 1  
**Product:** [Exact product name as the vendor uses it]  
**Vendor:** [Vendor or organisation name]  
**Versions covered:** [List each version — e.g. v6.7 · v6.8 · v7.0]  
**Domain:** [Domain name — e.g. gxp-lab-automation · financial-compliance · clinical-trials]  
**Guardrail profile:** [Profile name from your domain's GUARDRAIL_PROFILE.md]  
**Archetypes served:** [List — e.g. Validation Engineer · IT Admin · Procurement Evaluator]

**Companion capability skills:**
<!--
List the capability skills this product skill is designed to work with.
Use exact skill_id values (folder names).
-->
- `[capability-skill-id]` — [one line description of what it does]
- `[capability-skill-id]` — [one line description]

**Trigger phrases:**
<!--
List the phrases a user might say that should activate this skill.
Include full name, common abbreviations, version identifiers,
and question patterns practitioners actually use.
Aim for 8–15 phrases.
-->
```
"[product name]", "[common abbreviation]", "[version identifier]",
"[product name] [common task]", "[product name] [compliance topic]",
"[vendor name] [product type]"
```

---

## Disclaimer

<!--
Copy this disclaimer verbatim. Do not modify it.
-->

DISCLAIMER: This skill is an independent, community-maintained resource.
It is not affiliated with, endorsed by, or sponsored by [Vendor Name] or
any other organisation. The product name is used solely to identify the
system this skill supports. All vendor documentation used with this skill
must be obtained by the user under their own licence agreement with the
vendor. No vendor documentation is included in this skill file or in
this repository.

---

## ── CONFIGURE THIS SECTION — YOUR ENVIRONMENT ONLY ──

<!--
Define placeholder variables for the user to fill in.
Rules:
- UPPERCASE_WITH_UNDERSCORES naming
- One variable per line
- Square bracket description tells user what to enter
- Never populate with real values
- Add version-specific path variables for each version covered
-->

```
KNOWLEDGE_PATH:       [path to your document store root folder]
PATH_V[version1]:     [path to [version1] subfolder, or BLANK if not used]
PATH_V[version2]:     [path to [version2] subfolder, or BLANK if not used]
PATH_V[version3]:     [path to [version3] subfolder, or BLANK if not used]
KB_URL:               [your vendor knowledge base URL, or BLANK if not available]
KB_SEARCH:            [your vendor KB search endpoint, or BLANK if not available]
```

<!--
Add domain-specific config variables here if needed.
Example: SITE_NAME, SYSTEM_OWNER, REGULATORY_JURISDICTION
-->

## ── END CONFIGURATION — DO NOT EDIT BELOW THIS LINE ──

---

## Step 0 — Configuration check

<!--
This is the first action Claude takes. Do not change the logic.
You may change the user-facing message text to fit your domain.
-->

Before doing anything else, check the configuration block above.

- If KNOWLEDGE_PATH is BLANK and KB_URL is BLANK:
  Stop and ask:
  > "To assist you with [product name], I need your document store details.
  > Please provide your [document store type] path and/or your [vendor] support portal URL."

- If KNOWLEDGE_PATH is BLANK but KB_URL is configured:
  Proceed using KB only. Note to user:
  > "Your document store is not configured. I will use the [vendor] knowledge base only.
  > For grounded, version-specific responses, consider adding your documentation to the knowledge store."

- If KB_URL is BLANK but KNOWLEDGE_PATH is configured:
  Proceed using document store only. Note to user:
  > "Your vendor knowledge base URL is not configured. I will use your document store only."

- If all configured:
  Proceed silently to Step 1.

Never proceed with retrieval against an empty KNOWLEDGE_PATH.
Never fabricate paths, URLs, or document names.

---

## Step 1 — Version confirmation

<!--
Mandatory for all product skills. Adjust version list to match your product.
-->

Ask the user:

> "Which version of [product name] are you running?
> ([version1] / [version2] / [version3] / other)"

Map the answer to the correct PATH_V variable:
- [version1] → PATH_V[version1]
- [version2] → PATH_V[version2]
- [version3] → PATH_V[version3]
- Unknown or other → ask for clarification before proceeding

Store as session values:
- `[CONFIRMED_VERSION]` = confirmed version
- `[CONFIRMED_PATH]` = resolved PATH_V value

**Never retrieve documents from a path that does not match `[CONFIRMED_VERSION]`.
Cross-version retrieval is a framework violation.**

---

## Step 2 — Context declaration

<!--
Write this block into the conversation after version confirmation.
Field names must match CONTEXT_PROTOCOL.md exactly.
Do not rename, reorder, or omit mandatory fields.
Extension fields go after session_date.
-->

Write the following block visibly in the conversation:

```
[PRODUCT_CONTEXT]
framework_version:  1.0
skill_id:           [your-skill-folder-name]
product:            [exact product name]
vendor:             [vendor name]
version:            {confirmed version}
knowledge_path:     {resolved path}
kb_url:             {KB_URL or NONE}
kb_search:          {KB_SEARCH or NONE}
domain:             [domain name]
guardrail_profile:  [profile name]
archetype:          {ask if not stated — validation_engineer | it_admin | evaluator | other}
session_date:       {today's date — YYYY-MM-DD}
[/PRODUCT_CONTEXT]
```

<!--
Add domain extension fields here if defined in your guardrail profile.
Example:
gxp_regulation:     21_CFR_Part_11
-->

Ask the user to confirm the context block is correct before proceeding.
Correct any errors before continuing.

---

## Step 3 — Document retrieval

<!--
Define the lookup table: which documents to retrieve for which query types.
Be specific — name document types, not just "the documentation."
Adjust the table rows to match your domain's document types.
-->

When the user's query requires information from documentation, retrieve as follows.

Navigate to `[CONFIRMED_PATH]` from session context.

### Document lookup table

| Query or task type | Documents to retrieve |
|---|---|
| [query type 1] | [document type A] · [document type B] |
| [query type 2] | [document type C] |
| [query type 3] | [document type D] · [document type E] |
| [query type 4] | [document type F] |

<!--
Add rows for all major query types a user might bring.
-->

### File handling

- `.pdf` files → use pdf-reading skill
- `.docx` files → use file-reading skill (docx mode)
- `.xlsx` files → use file-reading skill (xlsx mode)
- `.txt` files → read directly

If a document is not found at `[CONFIRMED_PATH]`:
> "The expected document [document name] was not found at your configured path.
> Please verify the document exists in your knowledge store and the path is correct."

Do not attempt retrieval from a different version path.

### Knowledge base retrieval

Fetch from `KB_SEARCH` using query terms from the user's question.

Record for every article retrieved:
- Article URL or ID
- Article title
- Retrieval date (today)
- Last updated date (if shown on article)

Apply freshness guardrail per guardrail profile.

---

## Step 4 — Synthesis

<!--
Define how Claude composes its response from retrieved content.
The fabrication firewall and domain safety filter are mandatory.
Adjust archetype tone descriptions to match your domain archetypes.
-->

Compose responses using only retrieved content. Apply these rules in order:

### Fabrication firewall

If a claim cannot be grounded in a retrieved document or KB article:
```
[UNVERIFIED — no source found in configured knowledge store]
```
Write this marker immediately after the claim.
Suggest: "[domain-appropriate verification channel — e.g. vendor support / regulatory authority]"

Never omit the marker to improve readability.
Never state an unverifiable claim as fact.

### Domain safety filter

If retrieved content suggests bypassing, disabling, or circumventing
[list the domain integrity controls — e.g. audit trail, access controls, approval workflows]:

> "⚠ [Domain] safety flag: This suggestion conflicts with [control type]
> requirements under [regulatory basis]. It must not be applied without
> [required process — e.g. formal risk assessment and change control]."

### Archetype tone

<!--
Define how Claude's response style changes per archetype.
Adjust to match the archetypes in your domain.
-->

- **[Archetype 1]** → [tone and style description — e.g. technical, protocol-ready, document-section references]
- **[Archetype 2]** → [tone and style description — e.g. procedural, step-by-step, includes command syntax]
- **[Archetype 3]** → [tone and style description — e.g. comparative, requirement-framed, highlights vendor commitments]

---

## Step 5 — Citations

<!--
Use the standard framework citation format exactly.
Do not create variants.
-->

Cite every substantive claim. Format:

- Document store source:
  `[Source: {filename}, {version path}, §{section} or p.{page}]`

- Knowledge base article:
  `[KB: {article URL or ID}, retrieved {date}, last updated {date}]`

- Multiple sources:
  `[Sources: {doc} p.{n} · KB:{article ID}]`

- Unverifiable claim:
  `[UNVERIFIED — no source found in configured knowledge store]`

---

## Product knowledge

<!--
This section provides Claude with background understanding to contextualise
retrieved documents. It is NOT a citable source. Label it clearly.

Organise by topic, not by document. Suggested topics:
- Version landscape: what is significant about each version
- System architecture: components, roles, how they connect
- Data model or information hierarchy
- Compliance-critical configuration points
- Common failure modes, misconfigurations, inspection findings
- Integration touch points
- Upgrade patterns and risks

Write in plain, accurate prose. A validation engineer or IT admin
should recognise it as correct.

DO NOT make this section a substitute for retrieved documentation.
Specific values, steps, and thresholds must still come from retrieval.
-->

> **Note:** This section provides background context to help Claude retrieve
> and reason about documentation accurately. It is not a citable source.
> Specific configuration values, procedural steps, and compliance thresholds
> must be sourced from retrieved documentation.

### [Topic 1 — e.g. Version landscape]

[Write factual, domain-expert-level background knowledge here.]

### [Topic 2 — e.g. System architecture]

[Write factual, domain-expert-level background knowledge here.]

### [Topic 3 — e.g. Compliance-critical configuration]

[Write factual, domain-expert-level background knowledge here.]

### [Topic 4 — e.g. Common failure modes]

[Write factual, domain-expert-level background knowledge here.]

<!--
Add as many topic sections as needed.
There is no maximum, but keep each section focused.
-->

---

## Handoff signal

<!--
Define the handoff signal this skill writes when the user's request
moves into capability skill territory.
Adjust suggested_next values to match real skill_ids in the repository.
-->

When the user's request requires document authoring, evaluation, triage,
or audit, write this block:

```
[PRODUCT_HANDOFF]
from_skill:      [your-skill-folder-name]
context_passed:  [product] [version] — [domain] — context block written
output_summary:  {one sentence: what was established or produced}
suggested_next:  {appropriate capability skill_id}
reason:          {one sentence: why that skill is the right next step}
user_action:     Load the {suggested_next} skill to continue. Your context will carry over.
[/PRODUCT_HANDOFF]
```

### Example handoff signals

<!--
Write 2–3 example handoff signals for the most common transitions
from this product skill to a capability skill.
-->

**Example 1 — transitioning to document co-authoring:**
```
[PRODUCT_HANDOFF]
from_skill:      [your-skill-folder-name]
context_passed:  [product] [version] — [domain] — context block written
output_summary:  System architecture reviewed, audit configuration confirmed
suggested_next:  [co-author-skill-id]
reason:          User needs to author a [document type] — requires document capability skill
user_action:     Load the [co-author-skill-id] skill to begin. Your context will carry over.
[/PRODUCT_HANDOFF]
```

**Example 2 — transitioning to product evaluation:**
```
[PRODUCT_HANDOFF]
from_skill:      [your-skill-folder-name]
context_passed:  [product] [version] — [domain] — context block written
output_summary:  Product knowledge established for evaluation
suggested_next:  product-evaluation
reason:          User wants to compare this product against requirements
user_action:     Load the product-evaluation skill to begin scoring. Your context will carry over.
[/PRODUCT_HANDOFF]
```

---

## Guardrails

<!--
Implement all seven guardrails per the GUARDRAIL_FRAMEWORK.md.
Copy your domain's guardrail profile and add skill-specific additions.
Do not weaken any guardrail relative to the profile.
-->

**Guardrail profile applied:** [profile name]

1. **Source discipline**
   [Copy from guardrail profile. Add skill-specific sources if needed.]

2. **Fabrication firewall**
   Marker: `[UNVERIFIED — no source found in configured knowledge store]`
   Unverified channel: [domain-appropriate — e.g. vendor support / regulatory authority]

3. **Version discipline**
   Version variables: PATH_V[version1] · PATH_V[version2] · PATH_V[version3]
   Cross-version retrieval: framework violation — never permitted without user confirmation
   [Add any product-specific version discrepancy warnings.]

4. **Conflict surfacing**
   Precedence rule: [from guardrail profile]
   Recommended action: [from guardrail profile]
   [Add any product-specific known conflict patterns.]

5. **Domain safety filter**
   Integrity controls: [from guardrail profile]
   Regulatory basis: [from guardrail profile]
   Required process: [from guardrail profile]
   [Add any product-specific controls — e.g. specific configuration settings
   that must never be disabled in this product.]

6. **Freshness signalling**
   Threshold: [from guardrail profile]
   Flag format: "⚠ This article was last updated {date}. Verify it applies to {version}."

7. **Human decision gate**
   [From guardrail profile.]
   [Add any product-specific additional confirmation points.]
