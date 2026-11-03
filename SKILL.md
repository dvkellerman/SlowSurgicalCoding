---
name: slow-surgical-coding
description: "Implement features in deliberately small, human-approved stages instead of one large change: a Change Manifest followed by eight reviewable stages — scaffold, state, integration points, signatures, call sites, existing-type changes, bodies, verification — each ending with a review ritual and an explicit 'Awaiting approval for stage N' stop. Use when the user asks to work slowly or surgically, stage a feature, scaffold before implementing, review each step, or act as architect/reviewer; triggers include 'slow surgical coding', 'one stage at a time', 'go slow', 'small reviewable increments', 'don't implement it all at once', 'scaffold first', or 'staged implementation'."
---

## CRITICAL EXECUTION OVERRIDES
1. **EXECUTION_MODE:** `STRICT_INCREMENTAL_STATE_MACHINE`
2. **HARD TERMINATION RULE:** After printing the required Stage Output, end the turn IMMEDIATELY. Outputting text, code, or explanation for Stage N+1 in the same response as Stage N is a FATAL PROTOCOL VIOLATION.
3. **STRICT PREVENTATIVE BOUNDARIES:**
   - DO NOT write function bodies or logic before Stage 7.
   - DO NOT combine stages into a single response under any circumstance.
   - DO NOT proceed to the next stage unless the user prompt contains an explicit approval key (`approve`, `approve with notes`, `revise`, `replan`, `stop`).
4. **STATE CONTROL:** If the user prompt is anything other than explicit approval, treat it as feedback for the current stage, remain in the current stage, and re-issue the updated stage artifact with the review footer.

---

# Skill: Slow Surgical Coding

## Purpose

Implement features in deliberately small, reviewable, entertaining stages.
The agent must never rush from requirements to complete implementation.

The human is the architect and reviewer.
The agent is a careful junior engineer who proposes, explains, and waits.

## Absolute Rules

1. **Scaffold First:** Write only empty definitions in Stage 1. Implementation bodies are strictly forbidden prior to Stage 7.
2. **Single Layer Scope:** Modify exactly one anatomical layer per stage response.
3. **Mandatory Halts:** End every single stage output with the standardized **Review Footer** and halt output immediately.
4. **Discovered Flaws Protocol:** If a later stage reveals an issue in an earlier approved stage, HALT. Do not silently fix it. Issue a `REVISION` proposal to return to the earlier stage.
5. **Fail-Safe Artifacts:** Prefer compilation errors, TODOs, and commented-out call sites over incomplete or guessed logic.
6. **Review Bounds:** Keep every stage output small enough for human review in under 2–5 minutes.
7. **No Unrequested Tests:** Generate test cases only when explicitly requested by the human.
8. **Scope Isolation:** Do not refactor unrelated code.
9. **Zero Invention:** Do not invent requirements, APIs, naming, or architecture.
10. **Plan Adaptation:** If a task cannot safely fit this 8-stage pipeline, declare this immediately and propose a custom stage plan before producing code.

## Review & Approval Protocol

Every stage output MUST conclude with this exact structure:

```text
### Stage N: <name>

**What changed**
- <file>: <change>

**Why**
- <reason>

**What intentionally did not change**
- <list>

**Questions**
- <question or "none">

**Developer's Log**
> <1-2 sentences of flavor text>

**Next stage**
- <name and one-sentence description>

STATUS: WAITING_FOR_HUMAN_APPROVAL
STAGE: <N> (<Name>)
Awaiting approval for stage N: <name>.
```

*(Output MUST terminate immediately after printing `Awaiting approval for stage N: <name>.`)*

### Human Response Handling
- `approve` → Advance state to Stage N+1.
- `approve with notes: ...` → Advance state to Stage N+1 while applying notes.
- `revise: ...` → Re-generate Stage N incorporating requested fixes.
- `replan: ...` → Return to Change Manifest state.
- `stop` → Terminate workflow and print current state summary.

## Development Comment Format

Every new or changed declaration MUST include a structured `DEV` comment block:

```rust
/// DEV: [stage 2/state]
/// Purpose: caches the last validated user session.
/// Owner flow: refreshed by `SessionManager::refresh`, read by request guards.
/// Open question: should expired sessions be retained for audit?
/// TODO(stage 7): implement refresh and expiry logic.
pub last_valid_session: Option<Session>,
```

Required fields for all languages (using standard language comment syntax):
- `DEV`: `<stage_number>/<anatomical_layer>`
- `Purpose`: Brief explanation of existence.
- `Owner flow`: Creation, mutation, read, or cleanup path.
- `Open question`: Uncertainty requiring human resolution (or "None").
- `TODO(stage N)`: Targeted stage for completion.

---

## Stage Execution Pipeline

### Pre-Stage 0 — Change Manifest

Produce and submit the Change Manifest prior to any code generation:

```markdown
# Change Manifest: <feature name>

## Goal
<one paragraph>

## Non-goals
- <explicit exclusions>

## New entities
- <type/module/file>: <why>

## Changed entities
- <type/module/file>: <what changes and why>

## Removed entities
- <type/module/file>: <why removal is safe>

## External effects
- API, DB, config, protocol, UI, build, dependency changes

## Risks
- <risk>: <mitigation>

## Planned stages
1. Scaffold
2. State
3. Integration points
4. Signatures
5. Call sites
6. Existing-type changes
7. Bodies
8. Verification
```

*(Halt generation. Await manifest approval before proceeding to Stage 1.)*

---

### Stage 1 — Scaffold

**Goal:** Create structural skeletons with zero inner logic or fields.

**Allowed Content:**
- New files, empty modules, empty structs/classes/enums/traits/interfaces.
- Empty function signatures or placeholders where required to compile.
- `DEV` comments on every entity.

**Forbidden Content:** Fields, logic, call sites, or method bodies.

End with:
```text
SCAFFOLD MAP
- <entity>: <purpose>
- <entity>: <purpose>
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 2 — State

**Goal:** Declare fields, state structures, constants, and storage configurations.

**Allowed Content:**
- Fields and data structures with explicit `DEV` comments.
- Trivial constructors if required for compilation.

**Forbidden Content:** External behavior, non-trivial methods, or unapproved logic.

End with:
```text
STATE MAP
- Entity.field
  - type
  - initial value or uninitialized reason
  - writer(s)
  - reader(s)
  - lifetime / cleanup
  - risks
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 3 — Integration Points

**Goal:** Wire dependencies and define connection points.

**Allowed Content:**
- Imports, trait/interface declarations with empty bodies, registration points, dependency injection.
- Commented-out future call sites (e.g., `// TODO(stage 5): enable after X exists`).

End with:
```text
INTEGRATION MAP
- Existing component -> New component
  - direction of dependency
  - event / call / data flow
  - activation condition
  - current status: wired / commented / deferred
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 4 — Signatures

**Goal:** Define exact API contracts.

**Allowed Content:**
- Full function signatures with parameters and return types.
- Doc comments for contracts, preconditions, and errors.
- Method bodies containing strictly `todo!()`, `unimplemented!()`, or language equivalent.

End with:
```text
CONTRACT MAP
- fn name(args) -> result
  - purpose
  - inputs
  - outputs
  - errors
  - side effects
  - caller(s)
  - implementation stage
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 5 — Call Sites

**Goal:** Enable invocations of the new signatures in external call paths.

**Allowed Content:**
- Uncommenting and connecting Stage 3/4 call sites.
- Adding minimal call-site wiring and explicit error handling TODOs.

**Forbidden Content:** Surrounding business logic implementation.

End with:
```text
CALL-SITE MAP
- caller -> callee
  - trigger condition
  - arguments
  - expected result
  - failure behavior
  - current status
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 6 — Existing-Type Changes

**Goal:** Modify existing codebase types to accommodate new features.

**Allowed Content:**
- Added methods, trait implementations, field updates, or variant changes on existing types.
- Every modification must include a `DEV: [stage 6/existing-change]` block detailing `Before`, `After`, `Why`, and `Blast radius`.

End with:
```text
BLAST-RADIUS MAP
- Modified Entity
  - Changed aspect
  - Impacted callers
  - Mitigation / Verification path
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 7 — Bodies

**Goal:** Implement actual inner function logic in cohesive, minimal blocks.

**Rules:**
- Implement one cohesive group at a time.
- Stop and query the human if design ambiguities arise.
- Propose a `REVISION` if previous stages prove flawed.

For each body or group completed, present:
```text
BODY REPORT
- function
- approach
- edge cases handled
- edge cases deferred
- assumptions
- follow-up TODOs
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

### Stage 8 — Verification

**Goal:** Execute, validate, and verify the overall changes.

**Allowed Content:**
- Running compilation, linter passes, test suites, or manual verification steps.
- Providing proof of execution.

End with:
```text
VERIFICATION REPORT
- commands run
- expected result
- actual result
- failures and interpretation
- remaining risks
```
Followed by the standard **Review Footer**. `[HARD STOP]`

---

## Commit Ritual

1. **Base Commit:** Create the feature base commit after approval of Stage 1 (or initial agreed block).
2. **Fixup Commits:** Upon approval of each subsequent stage, commit changes as a fixup:
   `git commit --fixup <base-sha>`
   *Never mix multiple stages into one fixup.*
3. **No Rewrites:** Do not amend previously created commits during active staging. Post-approval revisions append a new fixup commit.
4. **Comment Cleanup:** `DEV:` markers remain during fixups and are removed during the final verification cleanup fixup.
5. **Final Squash:** Perform `git rebase -i --autosquash` onto the base commit prior to PR opening.
6. **Consent Boundary:** Confirm Git execution permissions before making commits.

---

## Completion Checklist

Before declaring the feature fully complete, confirm:
- [ ] Change Manifest aligns with final implementation.
- [ ] All new entities carry purpose documentation.
- [ ] State fields include lifecycle and ownership explanations.
- [ ] All TODOs are either completed or explicitly deferred.
- [ ] No unneeded commented-out code remains.
- [ ] Verification evidence is presented and human-approved.
