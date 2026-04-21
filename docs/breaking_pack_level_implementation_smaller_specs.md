I am providing you a finalized implementation details document.

Your job is NOT to rewrite, summarize, improve, or reinterpret it.

Your job is to perform a STRICT, LOSSLESS DECOMPOSITION of the document into multiple focused specs suitable for SpecKit.

CRITICAL REQUIREMENT (NON-NEGOTIABLE):

- You must NOT change ANY logic
- You must NOT rewrite ANY sentence
- You must NOT summarize ANY section
- You must NOT simplify ANY content
- You must NOT rephrase ANY rule
- You must NOT merge unrelated sections
- You must NOT introduce new structure beyond grouping

You are ONLY allowed to:
- group existing content into focused specs
- copy content exactly as-is into the appropriate spec

Think of this as:
👉 CUT + GROUP
NOT:
❌ REWRITE
❌ REGENERATE

---

# OBJECTIVE

Divide the implementation document into multiple focused specs where:

Each spec:
- owns a single clear system/module
- contains ONLY the content relevant to that system
- preserves ALL original wording exactly
- preserves ALL original sections exactly (no rewriting)
- preserves ALL rules, constraints, APIs, DB schema, flows exactly
- maintains execution-level detail
- is suitable for direct use by developers or coding agents

---

# SPEC FORMAT REQUIREMENT (VERY IMPORTANT)

Each focused spec MUST:

1. Follow the SAME STRUCTURE as the original implementation document
   (e.g., sections like Purpose, Scope, Roles, UI, Flows, HLD, LLD, DB, APIs, Rules, etc.)

2. Only include sections that are relevant to that system

3. If a section is partially relevant:
   - include ONLY the relevant parts
   - DO NOT rewrite them
   - DO NOT summarize them

4. If a section is fully relevant:
   - copy it entirely without modification

5. If a section belongs to another system:
   - DO NOT duplicate it
   - DO NOT redefine it

---

# SYSTEM IDENTIFICATION RULES

You must identify systems based on:
- ownership boundaries
- workflows
- state machines
- APIs
- data ownership
- responsibilities

Examples of valid system splits:
- Auth
- Verification
- User Management
- Enrollment
- Payment Reference
- Mapping
- Admin Operations
- Access Control
- Settings

Avoid:
- overly broad specs
- overly tiny specs
- mixing unrelated responsibilities

Target:
👉 8–15 focused specs (unless document strongly suggests otherwise)

---

# OUTPUT STRUCTURE

## Step 1: Recommended Spec Breakdown

For each spec:
- Spec Name
- Why it is a separate system
- What it owns

---

## Step 2: Focused Specs (LOSSLESS COPIES)

For each spec write its spec contents into a markdown file with a suitable name:

### Spec: [SPEC NAME]

Add a short header:

- This spec is derived from: [DOCUMENT NAME]
- This spec owns: [SYSTEM NAME]
- This spec strictly preserves original implementation details without modification

Then:

👉 Copy ONLY the relevant content from the implementation document
👉 Preserve EXACT wording
👉 Preserve EXACT formatting (headings, bullets, tables, code blocks)
👉 Preserve section hierarchy
👉 DO NOT rewrite anything

---

# STRICT CONTENT RULES

- If a line exists in the original doc → it must appear EXACTLY the same here if relevant
- Do NOT paraphrase
- Do NOT shorten
- Do NOT expand
- Do NOT normalize wording
- Do NOT convert into your own format
- Do NOT reinterpret meaning

---

# IMPORTANT EDGE RULES

- If content overlaps across systems:
  - place it in the most appropriate owner spec
  - do not duplicate unless absolutely necessary

- If a flow spans multiple systems:
  - include only the portion relevant to the spec
  - do not rewrite the flow

- If DB schema is shared:
  - include only relevant tables/columns per spec

- If APIs are shared:
  - include only endpoints owned by that spec

---

# FINAL GOAL

At the end, I should have:
- multiple focused specs
- each spec is a clean subset of the original document
- no loss of detail
- no change in behavior
- no ambiguity
- ready for direct engineering execution

---

Implementation document name:
[PUT_FILE_NAME_HERE]

Implementation document:
[PASTE_FULL_DOCUMENT_HERE]
