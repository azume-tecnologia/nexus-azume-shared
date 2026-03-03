---
allowed-tools: Bash, Read, Glob, Grep, AskUserQuestion, Edit, Write
argument-hint: <spec-path> ["custom-instructions"]
description: Review cross-project spec implementation for gaps, inconsistencies, bugs, and ship-readiness
---

# Review Spec Implementation Command

Review the implementation of a spec document for gaps, inconsistencies, bugs, logic flaws, and ship-readiness. The spec is the source of truth — audit production and test code against what the spec defines.

## Usage

```
/review-spec <spec-path> ["custom-instructions"]
```

**Arguments:**
1. **spec-path** (required): Path to the spec document (e.g., `specs/leitor-de-faturas-spec.md`)
2. **custom-instructions** (optional): Quoted scoping/focus instructions (e.g., `"Nexus Core only"`, `"Check wiring only"`)

**Examples:**
- `/review-spec specs/leitor-de-faturas-spec.md`
- `/review-spec specs/leitor-de-faturas-spec.md "Nexus Core only"`
- `/review-spec specs/leitor-de-faturas-spec.md "Azume Backend and Frontend"`
- `/review-spec specs/leitor-de-faturas-spec.md "Check wiring only"`

---

## Your Task

### Step 1: Parse Arguments & Read Spec

1. **Extract arguments from `$ARGUMENTS`:**
   - First argument: spec file path (required)
   - If the last argument is a quoted string, extract it as custom instructions (optional)
   - If no spec path provided: display error with usage examples and exit

2. **Read the spec file** using Read tool. If not found: `Error: Spec file not found at {path}` → exit

3. **Display confirmation:**
   ```
   Reviewing spec implementation:
   Spec: {spec_path}
   Custom Instructions: {custom_instructions or "None"}

   Reading specification...
   ```

### Step 2: Spec Analysis — Extract Expected Implementation

1. **Read the full spec document** and extract:
   - System inventory from Section 1 (Overview / sequence diagram)
   - Per-system file manifest from Sections 3–5+ (inline file references within each system section — section count may vary per spec)
   - API contracts from Section 2 (endpoints, request/response shapes, error codes)
   - Cross-system field mapping table from Section 2.4 (field names, types at each boundary)
   - Per-system test expectations from Section 7 (test matrices)
   - `<!-- Implementation Feedback -->` blocks (if present) — note what was already implemented and any issues

2. **Build a per-system manifest** of:
   - **Production files:** All files each system's spec section says to create or modify
   - **Test files:** All test files each system's spec section references
   - **Expected behaviors:** Key behaviors each file should implement (from spec descriptions)
   - **API contracts:** Cross-system endpoints and field mappings this system must honor

3. **Display manifest summary:**
   ```
   ## Spec Manifest

   | System | Repo Root | Production Files | Test Files |
   |--------|-----------|-----------------|------------|
   | Nexus Core | /home/paulo/projects/nexus/nexus-core | {count} | {count} |
   | Azume Backend | /home/paulo/projects/azume/azume-backend | {count} | {count} |
   | Azume Frontend | /home/paulo/projects/azume/azume-frontend-crm | {count} | {count} |

   **API boundaries:** {count} cross-system contracts
   **Field mappings:** {count} shared fields

   {If custom instructions apply, note which systems/sections are in scope}
   ```

### Step 3: Production Code Audit

**Audit each system in order** (skip systems excluded by custom instructions):
1. **Nexus Core** → `/home/paulo/projects/nexus/nexus-core`
2. **Azume Backend** → `/home/paulo/projects/azume/azume-backend`
3. **Azume Frontend** → `/home/paulo/projects/azume/azume-frontend-crm`

For each production file in the current system's manifest:

1. **Verify existence** using Glob. If missing: flag as critical gap.

2. **Read file content** and check against spec:

   **A. Gaps (missing implementations):**
   - Functions/classes defined in spec but not in code
   - Parameters/arguments specified in spec but missing
   - Return types or behaviors defined in spec but not implemented
   - Stub implementations (e.g., returns "not yet implemented", `pass`, `TODO`)

   **B. Inconsistencies (code vs spec mismatch):**
   - Different function signatures than spec defines
   - Different class names, method names, or parameter names
   - Different behavior than what spec describes
   - Logic that contradicts spec requirements

   **C. Bugs (logic errors, missing error handling):**
   - Off-by-one errors, null/None mishandling
   - Missing error handling for cases spec describes
   - Type mismatches, async/await mistakes
   - Edge cases described in spec but not handled

   **D. Wiring (integration completeness):**
   Per-stack checks — apply the relevant set based on current system:
   - **Nexus Core (Python/FastAPI):** Routes registered in router, services injected via `Depends()`, factories updated, `__init__.py` exports updated
   - **Azume Backend (TypeScript/Express):** Route file mounted in `app.ts`, middleware chain connected, imports resolve
   - **Azume Frontend (React/TypeScript):** Props wired per spec's component hierarchy, state threaded correctly, component imports resolve
   - **All systems:** Dependencies connected end-to-end per spec's architecture diagrams, cross-system API calls match contracts from Section 2

   **E. Business Logic Flaws (code works, but wrong business behavior):**

   Code that runs without errors but does not match the business intent described in the spec. These are subtle and dangerous — they pass tests if tests were written against the wrong assumption.

   - **Wrong sort/order:** Query fetches oldest-first when spec intends newest-first (e.g., chat messages, activity feeds)
   - **Wrong default behavior:** Default state or fallback that contradicts the feature's purpose (e.g., defaulting to "public" when spec says "private by default")
   - **Inverted conditions:** Filter includes when it should exclude, or vice versa (e.g., showing archived items in an "active" view)
   - **Wrong aggregation scope:** Counting/summing across all records when spec scopes to current user/tenant/session
   - **Missing business constraints:** No limit on items when spec implies pagination, no deduplication when spec implies uniqueness
   - **Wrong lifecycle/state transitions:** Allowing transitions the business flow forbids (e.g., reopening a resolved ticket when spec says only admins can)
   - **Mismatched units or formats:** Storing cents but displaying as dollars, using UTC when spec says user timezone

   **How to detect:** For each behavior, ask: "Does this do what a user/product owner expects?" — not just "Does this run without errors?" Re-read the spec's Context, Goal, and task descriptions to understand the *why* behind each implementation, then verify the code honors that intent.

   **If uncertain about business intent:** Use `AskUserQuestion` to ask the user. Do NOT guess or assume — a wrong assumption here creates a silent bug. Frame the question with the spec reference and what the code does, so the user can confirm or correct.

   **F. Ship-readiness flags:**
   - `TODO`, `FIXME`, `HACK` comments
   - Debug/print statements
   - Hardcoded values that should be configurable
   - Incomplete error handling (bare `except` in Python, empty `catch` blocks in TypeScript)
   - `assert` in production code — Python: should be `raise`; TypeScript: remove or replace with runtime validation

3. **Apply design principles** (stack-adaptive):
   - **Nexus Core:** Use the project's checklist at `/home/paulo/projects/nexus/nexus-core/.claude/checklists/coding-standards-checklist.md`
   - **Azume Backend / Frontend:** Apply general principles — Modularity, High Cohesion, Separation of Concerns, Abstraction, Loose Coupling (SRP, SoC)
   - Check for red flags: god classes, tight coupling, leaky abstractions, mixed concerns

4. **Categorize each finding by severity:**
   - **Critical:** Missing implementation, broken wiring, logic bugs, business logic flaws, security issues
   - **Important:** Spec inconsistencies, design violations, incomplete error handling
   - **Minor:** Naming mismatches, style issues, minor TODOs

### Step 4: Test Audit

For each test file in the manifest:

1. **Verify existence** using Glob. If missing: flag as critical gap.

2. **Read test content** and evaluate:

   **A. Coverage adequacy:**
   - All test cases listed in spec exist in test file
   - Happy path covered
   - Edge cases from spec covered
   - Error cases from spec covered

   **B. Test quality (per-stack):**
   - **Nexus Core (pytest):** BDD-style naming (`test_should_X_when_Y`), Arrange-Act-Assert pattern, one behavior per test, proper markers (`@pytest.mark.unit`, `@pytest.mark.integration`)
   - **Azume Backend (Jest):** `describe`/`it` blocks, clear test descriptions, one assertion focus per test
   - **Azume Frontend (React Testing Library):** User interaction tests, accessible queries (`getByRole`, `getByText`), async handling with `waitFor`/`findBy`

   **C. Mocking quality (per-stack):**
   - **Nexus Core:** Mocks YOUR protocols (not third-party libraries), uses `AsyncMock(spec=YourProtocol)`, no `@patch('openai.Client')`
   - **Azume Backend:** `jest.mock()` for modules, mock at network boundary not internal modules
   - **Azume Frontend:** Network-level mocks (MSW or fetch mocks), not internal implementation mocking

   **D. Test-production alignment:**
   - Tests match current production code (not stale)
   - Tests for new production code exist
   - Modified production code has updated tests

3. **Categorize test findings** using same severity levels as Step 3.

### Step 5: Automated Quality Checks

Run checks **per system** (skip systems excluded by custom instructions):

1. **Nexus Core:**
   ```bash
   cd /home/paulo/projects/nexus/nexus-core && make check
   cd /home/paulo/projects/nexus/nexus-core && make test-path TARGET={relevant_test_paths}
   ```

2. **Azume Backend:**
   ```bash
   cd /home/paulo/projects/azume/azume-backend && npm run build
   ```

3. **Azume Frontend:**
   ```bash
   cd /home/paulo/projects/azume/azume-frontend-crm && npm test -- --watchAll=false
   ```

4. **Parse output per system:**
   - Format/lint/type errors with file:line
   - Test pass/fail counts
   - Failing test names and error messages

5. **Display:**
   ```
   ## Automated Checks

   | System | Quality Check | Tests |
   |--------|--------------|-------|
   | Nexus Core | {Pass/Fail (count)} | {X passed, Y failed} |
   | Azume Backend | {Pass/Fail (count)} | — |
   | Azume Frontend | — | {X passed, Y failed} |
   ```

### Step 6: Assessment Report

Consolidate all findings into a structured report:

```markdown
# Spec Implementation Review

## Summary
**Spec:** {spec_path}
**Custom Instructions:** {custom_instructions or "None"}
**Ship-readiness:** {Ready / Not Ready}

## Spec Coverage
| System | Section | Implementation Status | Issues |
|--------|---------|----------------------|--------|
| {system} | {section} | {Complete/Partial/Missing} | {count or "None"} |
| ... | ... | ... | ... |

**Coverage:** {completed_sections}/{total_sections} sections fully implemented

## Issues by Category

### Gaps ({count})
{For each gap:}
**{file}:{line}** — {description}
Spec says: {what spec expects}
Fix: {suggestion}

### Inconsistencies ({count})
{For each inconsistency:}
**{file}:{line}** — {description}
Spec says: {what spec expects}
Code does: {what code actually does}
Fix: {suggestion}

### Bugs ({count})
{For each bug:}
**{file}:{line}** — {description}
Impact: {what could go wrong}
Fix: {suggestion}

### Wiring ({count})
{For each wiring issue:}
**{file}:{line}** — {description}
Expected: {what should be connected}
Fix: {suggestion}

### Business Logic Flaws ({count})
{For each flaw:}
**{file}:{line}** — {description}
Spec intent: {what the business behavior should be}
Code does: {what the code actually does — it works, but it's wrong}
Impact: {user-facing consequence}
Fix: {suggestion}

### Ship-readiness ({count})
{For each flag:}
**{file}:{line}** — {description}
Fix: {suggestion}

### Design ({count})
{For each design issue:}
**{file}:{line}** — {description}
System: {Nexus Core / Azume Backend / Azume Frontend}
Principle: {which of 5 principles}
Fix: {suggestion}

## Test Audit
**Test files:** {count existing}/{count expected}
**Test quality:** {Excellent/Good/Fair/Poor}
- BDD naming: {Pass/Fail}
- Arrange-Act-Assert: {Pass/Fail}
- Proper mocking: {Pass/Fail}
- Coverage adequacy: {Pass/Fail}
{For each test issue:}
**{file}:{line}** — {description}

## Automated Checks
- Format: {Pass / Fail (count)}
- Lint: {Pass / Fail (count)}
- Type: {Pass / Fail (count)}
- Tests: {X passed, Y failed}
{If failures, list them}

## Fix Plan
{If issues found, list fixes in priority order:}
1. **{severity}: {file}:{line}** — {description}
   - **Nexus Core:** RED: {test to write first} → GREEN: {minimal fix}
   - **Azume Backend/Frontend:** Verify with build or test run → {fix description}
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
    {"label": "Report only", "description": "Review complete, fix manually"},
    {"label": "Fix issues", "description": "Fix all found issues following TDD"},
    {"label": "Fix critical only", "description": "Fix only critical severity issues"}
  ]
}
```

**Handle response:**

**If "Report only":**
- Display summary, exit

**If "Fix issues" or "Fix critical only":**

For each issue (filtered by severity if "Fix critical only"), in priority order:

1. **Display the issue:**
   ```markdown
   ## Fix {n}/{total}: {category} — {severity}
   **File:** {file}:{line}
   **Issue:** {description}
   **RED (test):** {test to write/update}
   **GREEN (fix):** {implementation change}
   ```

2. **Ask approval:**
   ```javascript
   {
     "question": "Apply this fix?",
     "header": "Fix Issue",
     "options": [
       {"label": "Yes, apply fix"},
       {"label": "Skip this issue"},
       {"label": "Skip all remaining"}
     ]
   }
   ```

3. **If approved:**
   - Write/update test first (RED)
   - Implement the fix (GREEN)
   - Continue to next issue

4. **If "Skip":** Continue to next issue
5. **If "Skip all":** Exit fix loop

**After all fixes:**

1. Re-run per-system checks:
   - Nexus Core: `cd /home/paulo/projects/nexus/nexus-core && make check && make test-path TARGET={relevant_test_paths}`
   - Azume Backend: `cd /home/paulo/projects/azume/azume-backend && npm run build`
   - Azume Frontend: `cd /home/paulo/projects/azume/azume-frontend-crm && npm test -- --watchAll=false`
2. Display fix summary:
   ```markdown
   ## Fix Summary
   **Fixed:** {count}
   **Skipped:** {count}
   **Modified files:** {list, grouped by system}
   **Per-system checks:**
   | System | Quality | Tests |
   |--------|---------|-------|
   | Nexus Core | {Pass/Fail} | {X passed, Y failed} |
   | Azume Backend | {Pass/Fail} | — |
   | Azume Frontend | — | {X passed, Y failed} |

   ## Next Steps
   1. Review changes in each repo: `cd {repo_root} && git diff`
   2. Commit in each affected repo using that repo's standard commit flow
   ```

---

## Guidelines

**Running Commands:**
- Run commands from each system's repo root using absolute paths:

  | System | Repo Root | Quality Command | Test Command |
  |--------|-----------|----------------|-------------|
  | Nexus Core | `/home/paulo/projects/nexus/nexus-core` | `make check` | `make test-path TARGET=...` |
  | Azume Backend | `/home/paulo/projects/azume/azume-backend` | `npm run build` | — |
  | Azume Frontend | `/home/paulo/projects/azume/azume-frontend-crm` | — | `npm test -- --watchAll=false` |

**Review Philosophy:**
- **Spec is source of truth** — if the spec says it, the code must do it
- Spec-driven review (audits code against the spec as source of truth)
- Read-only by default — fixes require explicit user approval at the action gate
- TDD-oriented fixes: every fix includes RED (test) then GREEN (implementation)
- Custom instructions enable scoping (e.g., "Nexus Core only", "Azume Backend and Frontend", "Check wiring only")

**Ask for Clarification — Do NOT Guess:**
- Use `AskUserQuestion` whenever the spec is ambiguous, underspecified, or you are unsure about business intent
- Common situations requiring clarification:
  - Spec describes a behavior but the intended sort order, default value, or scope is unclear
  - Code does something reasonable but the spec could be interpreted multiple ways
  - A business logic flaw is suspected but you are not 100% certain of the intended behavior
  - Spec references external context (other specs, existing features) that you cannot fully verify
- Frame questions clearly: quote the spec section, describe what the code does, and ask which interpretation is correct
- **Never flag a business logic flaw as confirmed if you are uncertain** — ask first, then categorize based on the answer

**Safety:**
- Never auto-fix without approval
- Show issue context before applying fixes
- Re-run checks after modifications
- Git status check before/after fixes

**Output Requirements:**
- Specific: file:line, quoted code, before/after, spec references
- Actionable: explain what's wrong, why, and how to fix
- Prioritized: critical → important → minor
- Positive feedback: acknowledge good practices and completed spec tasks

## Notes

- **Multi-repo model:** Specs in this repo (`shared`) describe cross-project features spanning Nexus Core, Azume Backend, and Azume Frontend. This command navigates to each repo using absolute paths to audit implementation.
- **Spec format:** Numbered sections by system (Section 1: Overview, Section 2: API Contracts, Sections 3–5: per-system implementation, Section 6: reference data/enums, Section 7: verification & testing), cross-system field mapping tables, inline file annotations, Section 1 sequence diagram
- Respects `<!-- Implementation Feedback -->` blocks as prior implementation context
- Main agent (no subagent) — matches other review commands
- For Nexus Core design evaluation, uses the project's checklist at `/home/paulo/projects/nexus/nexus-core/.claude/checklists/coding-standards-checklist.md`; for Azume projects, applies general design principles
