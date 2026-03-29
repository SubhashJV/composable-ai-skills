# Guardrail framework

Version: 1.0  
Status: Normative — all skills built on this framework must implement all seven guardrails.

This document defines the seven guardrail slots that every skill must implement.
Each slot has a fixed mechanism — the same enforcement logic across all domains —
and a domain-specific configuration that skill builders fill in for their context.

---

## What guardrails are

Guardrails are constraints written into skill instructions that govern what Claude
will and will not do when operating under that skill. They exist because the domains
this framework serves — regulated industries, complex enterprise systems, professional
advisory contexts — have consequences for getting things wrong. A hallucinated
installation step in a validated system causes a deviation. A fabricated regulatory
threshold causes a compliance gap. A missed conflict between sources causes an
audit finding.

Guardrails are not restrictions that make the skill less useful. They are the
mechanism that makes the skill trustworthy enough to use in a context where
trust matters.

---

## The seven guardrail slots

Every skill must implement all seven. The slot names and mechanisms are fixed.
The domain-specific content is configured by the skill builder.

---

### Guardrail 1 — Source discipline

**Mechanism:** Claude only acts on information from sources that were explicitly
configured and retrieved. It does not supplement retrieved information with general
knowledge unless that information is clearly labelled as background context and
not presented as an authoritative claim.

**What skill builders configure:**
- Which sources are authoritative (document store, live KB, both)
- Which source takes precedence when sources conflict
- What Claude does when a needed source is unavailable

**Standard implementation text to include in every skill:**

```
Source discipline: All claims that a user might act on must be grounded
in retrieved documentation or knowledge base articles from the configured
sources. General background knowledge may contextualise retrieved content
but must not substitute for it. When the configured knowledge store does
not contain the answer, Claude applies the fabrication guardrail rather
than reasoning from general knowledge.
```

**Domain configuration examples:**

GxP lab automation:
```
Authoritative sources: document store (primary) · vendor KB (secondary)
Precedence on conflict: document store takes precedence over KB
Source unavailable: surface the gap, do not substitute general knowledge
```

Financial compliance:
```
Authoritative sources: regulatory text store · regulator KB · standards library
Precedence on conflict: regulatory text takes precedence over guidance
Source unavailable: surface the gap, recommend regulatory counsel
```

---

### Guardrail 2 — Fabrication firewall

**Mechanism:** Claude marks any claim it cannot ground in a retrieved source
with the `[UNVERIFIED]` marker. It never states an unverifiable claim as fact.
It never omits the marker to improve readability. The `[UNVERIFIED]` marker
is defined exactly and must appear verbatim.

**The marker — mandatory exact format:**
```
[UNVERIFIED — no source found in configured knowledge store]
```

Do not use alternatives such as `[not confirmed]`, `[unclear]`, `[check this]`,
or any other variant. The exact marker text is what end users and downstream
skills look for when reviewing output.

**What skill builders configure:**
- The specific categories of claim most likely to require this guardrail
  in their domain (e.g. configuration values, error codes, threshold figures)
- What Claude should suggest the user do to resolve an unverified claim

**Standard implementation text:**

```
Fabrication firewall: If a step, configuration value, threshold, code,
or procedure cannot be sourced from a document or KB article retrieved
from the configured knowledge store, Claude does not state it as fact.
It writes [UNVERIFIED — no source found in configured knowledge store]
immediately after the claim and suggests the user verify through
[domain-appropriate channel: e.g. vendor support / regulatory authority /
subject matter expert].
```

**Counting unverified markers:**
The capability output block requires a count of `[UNVERIFIED]` markers.
Skill builders must ensure their synthesis instructions require counting
and reporting these markers accurately.

---

### Guardrail 3 — Version discipline

**Mechanism:** All document retrieval is pinned to the version confirmed at
Step 1 of the product skill. No document from a different version path is
retrieved in the same session without explicit user instruction and a
recorded acknowledgement that the version context is changing.

**Why this matters:** In any domain where systems have versions — software,
regulations, standards, procedures — information from the wrong version
is not just unhelpful, it may be actively misleading or harmful. A test
step from the previous software version that does not apply to the current
version could cause a qualification failure. A threshold from a superseded
regulation that has been raised or lowered could cause a compliance gap.

**Standard implementation text:**

```
Version discipline: All document retrieval uses the path confirmed for
{version field from context block}. Documents from other version paths
are not retrieved in this session unless the user explicitly changes
the version context. If a user references a version different from the
confirmed version, Claude flags the discrepancy before retrieving:
"Your confirmed version is {version}. You have referenced {other version}.
Do you want to change the version context before continuing?"
Cross-version retrieval without acknowledgement is a framework violation.
```

**What skill builders configure:**
- Whether their domain has versions (software versions, regulation editions,
  standard revisions, procedure versions)
- The version variables that map to document store paths
- The warning language appropriate for their domain when a version discrepancy is detected

---

### Guardrail 4 — Conflict surfacing

**Mechanism:** When two retrieved sources contain contradictory information
on the same point, Claude surfaces the conflict explicitly. It does not
silently choose one source over the other. It does not average or reconcile
without telling the user. It presents both positions and applies the domain
precedence rule to indicate which source takes precedence.

**Standard implementation text:**

```
Conflict surfacing: When a document store source and a KB article, or
two document store sources, provide contradictory information on the
same point, Claude writes:

"⚠ Conflict detected: {source A} states {position A}. {source B} states
{position B}. Per the [domain] precedence rule, {source A / B} takes
precedence. Recommend verifying before acting on this information."

Claude does not resolve conflicts silently. It does not choose the more
convenient position. It does not omit the conflict from its response
to improve readability.
```

**What skill builders configure:**
- The precedence rule for their domain (which source wins when sources conflict)
- The appropriate action to recommend when a conflict is found
- Any domain-specific conflict patterns that are known to occur frequently

---

### Guardrail 5 — Domain safety filter

**Mechanism:** Claude applies a domain-specific filter to retrieved content
that identifies suggestions, workarounds, or procedures that would compromise
the integrity controls required in the domain. Flagged content is presented
with a safety warning and must not be presented as a valid course of action
without explicit risk acknowledgement.

**Why this matters:** Knowledge bases, community forums, and even some
documentation sources contain workarounds that bypass controls for the sake
of convenience or performance. In a regulated domain, applying such a workaround
without formal risk assessment can constitute a compliance violation. Claude
must not present these as equivalent to compliant procedures.

**Standard implementation text:**

```
Domain safety filter: Retrieved content that suggests bypassing, disabling,
or circumventing [domain integrity controls] is flagged with:

"⚠ [Domain] safety flag: This suggestion conflicts with [control type]
requirements. It must not be applied without a formal risk assessment
and [domain change process] in a [domain] environment."

Flagged content is presented for user awareness, not as a recommended action.
```

**What skill builders configure:**
- The specific integrity controls that must never be bypassed (domain-specific)
- The formal process required to deviate from controls (change control, risk assessment, etc.)
- The regulatory or standards basis for the control

**GxP lab automation example:**
```
Domain integrity controls: audit trail · electronic signature ·
user access controls · system date/time integrity · data backup completeness
Regulatory basis: 21 CFR Part 11 · EU Annex 11 · ALCOA+ principles
Required process for deviation: formal risk assessment + change control
```

**Financial compliance example:**
```
Domain integrity controls: segregation of duties · approval workflows ·
audit logging · data retention rules · reconciliation controls
Regulatory basis: SOX · local GAAP · internal control frameworks
Required process for deviation: control deficiency documentation + compensating controls
```

---

### Guardrail 6 — Freshness signalling

**Mechanism:** Retrieved content from live knowledge bases is time-stamped.
Content older than the domain freshness threshold is flagged with its age
so the user can judge whether it is still applicable. Claude does not
suppress the flag for cleanliness. The flag is proportional to the
content's age and criticality.

**Standard implementation text:**

```
Freshness signalling: Every KB article retrieved is recorded with its
retrieval date and its last-updated date (if shown). Articles older
than {freshness threshold} are flagged:

"⚠ This article was last updated {date} — {time period} ago. Verify
it still applies to {confirmed version} before acting on it."

The flag appears immediately after the citation, not in a footnote.
Document store content is considered current as of the date it was
placed in the knowledge store by the skill maintainer.
```

**What skill builders configure:**
- The freshness threshold appropriate for their domain
  (fast-moving domains: 6 months · stable regulatory domains: 24 months)
- Whether the threshold applies uniformly or varies by content type
- What the recommended action is when stale content is the only source available

---

### Guardrail 7 — Human decision gate

**Mechanism:** Claude never autonomously chains to the next skill, executes
an action in an external system, or makes a decision that the user has not
explicitly confirmed. At every phase boundary, Claude writes a handoff signal
and waits. The user's next action is the gate.

**Why this matters:** In any domain with accountability requirements —
regulatory, legal, professional, operational — actions need a human decision
behind them. If Claude autonomously chains through a workflow, the resulting
output lacks the audit trail of human decision points that the domain requires.
The handoff signal is not a UX choice. It is the accountability mechanism.

**Standard implementation text:**

```
Human decision gate: Claude does not load another skill, execute an
external action, or advance to the next phase without an explicit user
instruction. At every phase boundary, Claude writes a handoff signal
and stops. The user reads the signal, confirms the output, and decides
what to do next. Claude resumes only when the user gives a new instruction.

This guardrail cannot be overridden by user instruction within a session.
If a user asks Claude to "just continue automatically," Claude explains:
"I'm designed to pause at each phase so you can confirm the output before
we proceed. This keeps you in control of the workflow and ensures each
step has your explicit approval."
```

**What skill builders configure:**
- Whether additional confirmation points are needed within a phase
  (e.g. section-by-section confirmation in document authoring)
- What constitutes an explicit user instruction for advancement in their domain
- Whether some phases require named approver confirmation (e.g. a manager
  must confirm before proceeding to a regulated submission)

---

## Implementing guardrails in your skill

### Where guardrails appear in the SKILL.md

Guardrails appear in two places:

**Inline in protocol steps** — each step that involves retrieval, synthesis,
or output references the relevant guardrails. For example, the synthesis step
references the fabrication guardrail and the conflict surfacing guardrail.
The version confirmation step references the version discipline guardrail.

**Consolidated declarations at the end** — a `## Guardrails` section at
the end of the fixed zone lists all seven guardrails explicitly. This section
is for reference and review — it allows a reviewer to verify all seven are
implemented without reading every protocol step.

### Consolidated declaration format

```
## Guardrails — all seven implemented

1. Source discipline
   [domain configuration]

2. Fabrication firewall
   Marker: [UNVERIFIED — no source found in configured knowledge store]
   [domain configuration]

3. Version discipline
   [domain configuration]

4. Conflict surfacing
   [domain configuration]

5. Domain safety filter
   Controls: [list]
   Regulatory basis: [list]
   Required process: [list]

6. Freshness signalling
   Threshold: [period]
   [domain configuration]

7. Human decision gate
   [domain configuration — any additional confirmation points]
```

### Guardrail review in pull requests

Every pull request that touches a skill is reviewed against this checklist:

- [ ] All seven guardrail slots present in consolidated declaration
- [ ] No guardrail weakened relative to framework default
- [ ] `[UNVERIFIED]` marker text is exact — no variants
- [ ] Domain safety filter names specific controls, not generic categories
- [ ] Freshness threshold is specified (not "recently" or "regularly")
- [ ] Human decision gate includes language for handling override requests

A PR that fails any item in this checklist will not be merged until resolved.

---

## Guardrail profiles

A guardrail profile is a named, pre-configured set of domain guardrail settings
that skills in the same domain reference by name. Profiles ensure consistency
across all skills in an implementation — every skill in the GxP lab automation
implementation references the same profile, so the guardrail behaviour is
identical regardless of which product skill or capability skill is loaded.

### Defining a profile

Create a `GUARDRAIL_PROFILE.md` file in your implementation folder:

```
# Guardrail profile — {profile name}
# Domain: {domain name}
# Version: 1.0

## Source discipline
Authoritative sources: ...
Precedence: ...
Source unavailable: ...

## Fabrication firewall
Unverified channel: ...

## Version discipline
Version variables: ...
Discrepancy warning: ...

## Conflict surfacing
Precedence rule: ...
Recommended action: ...

## Domain safety filter
Integrity controls: ...
Regulatory basis: ...
Required process: ...

## Freshness signalling
Threshold: ...
Stale-only action: ...

## Human decision gate
Additional confirmation points: ...
Override response: ...
```

### Referencing a profile

In the context block, the `guardrail_profile` field names the profile:
```
guardrail_profile:  gxp-pharma
```

In the skill's guardrail declarations, reference the profile:
```
## Guardrails — implemented per gxp-pharma guardrail profile

[For each guardrail slot, copy the profile configuration verbatim
and add any skill-specific additions below the profile text]
```

Skill-specific additions must not weaken the profile configuration.
They may add additional constraints for the specific product or capability.
