---
allowed-tools: Read, Glob, Grep, AskUserQuestion, Edit, Write
argument-hint: <spec-path> ["custom-instructions"]
description: Review a planned spec for gaps, inconsistencies, logic flaws, and completeness before implementation
---

# Review Spec Plan Command

Review a spec document **before implementation** for gaps, inconsistencies, ambiguities, logic flaws, and completeness. The goal is to improve the spec itself so it is clear, correct, and implementation-ready. **This command NEVER implements the spec — the only deliverable is an improved spec document.**

## Usage

```
/review-spec-plan <spec-path> ["custom-instructions"]
```

**Arguments:**
1. **spec-path** (required): Path to the spec document (e.g., `specs/leitor-de-faturas-spec.md`)
2. **custom-instructions** (optional): Quoted scoping/focus instructions (e.g., `"Nexus Core sections only"`, `"Check data model only"`)

**Examples:**
- `/review-spec-plan specs/leitor-de-faturas-spec.md`
- `/review-spec-plan specs/leitor-de-faturas-spec.md "Nexus Core sections only"`
- `/review-spec-plan specs/leitor-de-faturas-spec.md "Check data model only"`

---

## Your Task

**CRITICAL CONSTRAINT:** You are reviewing and improving the spec document ONLY. You must NEVER create, modify, or generate any production code, test code, or any file other than the spec itself. The ONLY deliverable is the improved spec.

### Step 1: Parse Arguments & Read Spec

1. **Extract arguments from `$ARGUMENTS`:**
   - First argument: spec file path (required)
   - If the last argument is a quoted string, extract it as custom instructions (optional)
   - If no spec path provided: display error with usage examples and exit

2. **Read the spec file** using Read tool. If not found: `Error: Spec file not found at {path}` → exit

3. **Display confirmation:**
   ```
   Reviewing spec plan:
   Spec: {spec_path}
   Custom Instructions: {custom_instructions or "None"}

   Reading specification...
   ```

### Step 2: Spec Comprehension — Deep Read

Read the full spec document and build a mental model of:

1. **Context & Goals:**
   - What problem does this spec solve?
   - What is the business motivation?
   - Who are the users/consumers of this feature?

2. **Structure & Systems:**
   - All system sections (e.g., "Section 3: Nexus Core", "Section 4: Azume Backend", "Section 5: Azume Frontend")
   - Cross-system dependencies and API boundaries
   - Per-system file manifests from inline file references within each system section

3. **Technical Design:**
   - Data models, schemas, types
   - API contracts (endpoints, request/response shapes)
   - Architecture decisions (patterns, protocols, dependency flow)
   - Integration points with existing codebase

4. **Behavioral Specifications:**
   - Expected behaviors and outcomes
   - Edge cases and error handling
   - Business rules and constraints
   - Default values, fallbacks, limits

5. **Display comprehension summary:**
   ```
   ## Spec Comprehension

   **Goal:** {one-sentence summary of what this spec achieves}
   **Systems involved:** {list — e.g., Nexus Core, Azume Backend, Azume Frontend}
   **API boundaries:** {list of cross-system contracts — e.g., "Nexus → Azume Backend: POST /api/..."}
   **Key components:** {list of main models/services/endpoints per system}
   **Integration points:** {what existing code this touches in each system}

   {If custom instructions apply, note which systems/sections are in scope}
   ```

### Step 3: Spec Analysis — Check for Issues

Analyze the spec against the following categories:

**A. Gaps (missing or underspecified):**
- Missing error handling specifications (what happens when X fails?)
- Missing edge cases (empty inputs, max limits, concurrent access)
- Missing validation rules for inputs/parameters
- Undefined default values or fallback behaviors
- Missing lifecycle/cleanup considerations
- Vague descriptions that leave implementation ambiguous (e.g., "handle errors appropriately" without specifying how)
- Missing type definitions or incomplete schemas
- Undefined API response shapes for error cases

**B. Inconsistencies (contradictions within the spec):**
- Type definitions that contradict usage elsewhere in the spec
- Conflicting behavior descriptions across different sections
- Parameter names or signatures that differ between the spec's own sections
- Section ordering that conflicts (System A depends on output from System B, but System A's section is specified without that dependency)
- A section referencing a new file but a later section assumes it already exists

**C. Business Logic Flaws (spec describes wrong behavior):**
- Default values that contradict the feature's purpose
- Sort orders or priorities that don't match the use case
- Scope mismatches (per-user vs global, per-session vs per-document)
- Missing business constraints (no pagination, no deduplication, no rate limiting when the domain requires it)
- State transitions that allow invalid flows
- Conditions that are inverted from what the business intent suggests

**D. Architectural Issues (design problems):**
- API contract mismatches between systems (e.g., Nexus exposes field X but Backend expects field Y)
- Missing error propagation across system boundaries
- Responsibility boundary violations (one system doing another system's job)
- Circular or undefined data flow between systems
- Missing abstraction layers or direct cross-system coupling
- Services with too many responsibilities within a single system

**E. Ambiguities (multiple valid interpretations):**
- Descriptions that could be interpreted in more than one way
- Unspecified behavior for boundary conditions
- References to external specs or features without enough context
- Unclear ownership of cross-cutting concerns

**F. Completeness (spec readiness for implementation):**
- Are all sections actionable? Could a developer implement each section from the spec alone?
- Are test scenarios specified or at least implied for each behavior?
- Are migration/rollback considerations addressed if needed?
- Does the spec cover observability (logging, metrics) if appropriate?

### Step 4: Clarification Questions

If any findings from Step 3 require user input to resolve (ambiguities, uncertain business intent, unclear scope):

Use **AskUserQuestion** to ask the user. Frame questions clearly:
- Quote the relevant spec section
- Describe what's unclear or what the possible interpretations are
- Ask which interpretation is correct or what the intended behavior should be

**Do NOT guess or assume — ask first, then incorporate the answer into the improvement plan.**

Group related questions into a single AskUserQuestion call when possible (up to 4 questions per call).

### Step 5: Assessment Report

Consolidate all findings into a structured report:

```markdown
# Spec Plan Review

## Summary
**Spec:** {spec_path}
**Custom Instructions:** {custom_instructions or "None"}
**Implementation Readiness:** {Ready / Needs Improvements}

## Spec Quality
| Category | Issues Found |
|----------|-------------|
| Gaps | {count} |
| Inconsistencies | {count} |
| Business Logic Flaws | {count} |
| Architectural Issues | {count} |
| Ambiguities | {count} |
| Completeness | {count} |

## Findings

### Gaps ({count})
{For each gap:}
**{spec-section}** — {description}
Spec says: {what the spec currently states or omits}
Should specify: {what information is missing}
Severity: {Critical / Important / Minor}

### Inconsistencies ({count})
{For each inconsistency:}
**{spec-section}** — {description}
Section A says: {quote from one part}
Section B says: {quote from conflicting part}
Resolution: {suggested fix}
Severity: {Critical / Important / Minor}

### Business Logic Flaws ({count})
{For each flaw:}
**{spec-section}** — {description}
Spec intent: {what the feature should achieve}
Spec says: {what the spec actually describes}
Problem: {why this is wrong for the business case}
Suggestion: {corrected behavior}
Severity: {Critical / Important / Minor}

### Architectural Issues ({count})
{For each issue:}
**{spec-section}** — {description}
Principle: {which principle is violated — Modularity, Cohesion, SoC, Abstraction, Loose Coupling}
Suggestion: {how to fix the design in the spec}
Severity: {Critical / Important / Minor}

### Ambiguities ({count})
{For each ambiguity:}
**{spec-section}** — {description}
Possible interpretations: {list them}
Recommendation: {which interpretation to clarify in the spec, or user's answer from Step 4}
Severity: {Critical / Important / Minor}

### Completeness ({count})
{For each completeness issue:}
**{spec-section}** — {description}
Missing: {what should be added}
Severity: {Critical / Important / Minor}

## Strengths
{Acknowledge what the spec does well — clear sections, good structure, thorough coverage, etc.}
```

### Step 6: Improvement Plan

Based on the findings, create a prioritized plan of spec edits:

```markdown
## Improvement Plan

{Numbered list of spec changes, ordered by severity (critical first):}

1. **{severity}: {category}** — {one-line description}
   - Section: {which spec section to edit}
   - Change: {what to add/modify/remove in the spec}

2. ...
```

### Step 7: Action Gate

Use **AskUserQuestion**:

```javascript
{
  "question": "How would you like to proceed?",
  "header": "Next Steps",
  "multiSelect": false,
  "options": [
    {"label": "Report only", "description": "Review complete, I will edit the spec manually"},
    {"label": "Apply all improvements", "description": "Apply all planned improvements to the spec"},
    {"label": "Apply critical only", "description": "Apply only critical severity improvements to the spec"}
  ]
}
```

**Handle response:**

**If "Report only":**
- Display summary, exit

**If "Apply all improvements" or "Apply critical only":**

For each improvement (filtered by severity if "Apply critical only"), in priority order:

1. **Display the improvement:**
   ```markdown
   ## Improvement {n}/{total}: {category} — {severity}
   **Section:** {spec section}
   **Issue:** {description}
   **Change:** {what will be added/modified/removed in the spec}
   ```

2. **Apply the edit** to the spec document using Edit tool.

3. After all improvements are applied, display:
   ```markdown
   ## Improvement Summary
   **Applied:** {count}
   **Skipped:** {count}
   **Spec file modified:** {spec_path}

   ## Next Steps
   1. Review changes: `git diff {spec_path}`
   2. When ready to implement: open the spec in each affected project's Claude session and implement the relevant sections there
   ```

**REMINDER: The ONLY file modified is the spec document itself. No production code, no test code, no other files.**

---

## Guidelines

**Scope — Spec Document Only:**
- This command reviews and improves the SPEC DOCUMENT, not code
- The only file that may be edited is the spec file itself
- NEVER create, modify, or generate production code, test code, or any other file
- The deliverable is an improved, implementation-ready spec

**Review Philosophy:**
- Evaluate the spec as a blueprint — is it clear enough for a developer to implement without guessing?
- Check internal consistency — does the spec contradict itself anywhere?
- Check design principles — does the spec honor cohesion, separation of concerns, and loose coupling?
- Check cross-system consistency — are field names, types, and error codes consistent at every API boundary?
- Think like an implementer — what questions would you have if you were tasked with building this?

**Ask for Clarification — Do NOT Guess:**
- Use `AskUserQuestion` whenever the spec is ambiguous or you are unsure about business intent
- Common situations requiring clarification:
  - The spec describes a behavior but the intended outcome is unclear
  - A business rule could be interpreted multiple ways
  - The spec references external context you cannot verify
  - A design decision has trade-offs the user should weigh in on
- Frame questions clearly: quote the spec section, describe the ambiguity, and present the options
- **Never assume business intent** — ask first, then improve based on the answer

**Codebase Context:**
- Use Glob and Grep (read-only) to understand existing code structure when needed to evaluate the spec's design
- Check that file paths in the spec are consistent with the actual project structure
- Verify that existing models, services, or protocols referenced by the spec actually exist
- This read-only exploration informs the review — it does NOT lead to code changes
- **Cross-repo access:** Specs reference code across 3 repos. Use absolute paths to read files:
  - Nexus Core: `/home/paulo/projects/nexus/nexus-core`
  - Azume Backend: `/home/paulo/projects/azume/azume-backend`
  - Azume Frontend: `/home/paulo/projects/azume/azume-frontend-crm`

**Safety:**
- Never auto-apply improvements without approval at the action gate
- Show the full report before asking to proceed
- Only edit the spec file, never any other file

**Output Requirements:**
- Specific: quote spec sections, reference line numbers when possible
- Actionable: explain what's wrong, why, and what the spec should say instead
- Prioritized: critical → important → minor
- Positive feedback: acknowledge what the spec does well

## Notes

- Complements `/review-spec-implementation` (which reviews code AFTER implementation — both commands run from `shared`, navigating to other repos via absolute paths)
- Intended to run BEFORE implementation to ensure the spec is solid
- Does NOT generate code, does NOT run tests
- Main agent (no subagent) — matches other review commands
- Spec format expected: numbered sections by system (Section 1: Overview, Section 2: API Contracts, Sections 3–5: per-system implementation, Section 6: reference data/enums, Section 7: verification & testing), cross-system field mapping tables, inline file annotations, Section 1 sequence diagram
