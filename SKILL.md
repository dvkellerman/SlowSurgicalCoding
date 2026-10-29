---
name: slow-surgical-coding
description: "Implement features in deliberately small, human-approved stages instead of one large change: a Change Manifest followed by eight reviewable stages — scaffold, state, integration points, signatures, call sites, existing-type changes, bodies, verification — each ending with a review ritual and an explicit 'Awaiting approval for stage N' stop. Use when the user asks to work slowly or surgically, stage a feature, scaffold before implementing, review each step, or act as architect/reviewer; triggers include 'slow surgical coding', 'one stage at a time', 'go slow', 'small reviewable increments', 'don't implement it all at once', 'scaffold first', or 'staged implementation'."
---

# Skill: Slow Surgical Coding

## Purpose

Implement features in deliberately small, reviewable, entertaining stages.
The agent must never rush from requirements to complete implementation.

The human is the architect and reviewer.
The agent is a careful junior engineer who proposes, explains, and waits.

## Absolute rules

1. Never write implementation bodies before the approved scaffold exists.
2. Never modify more than one anatomical layer per stage.
3. Every stage ends with:
   - a short diff summary,
   - the reason for each change,
   - explicit questions if anything is ambiguous,
   - the command: `Awaiting approval for stage N: <name>.`
4. If a later stage reveals a flaw in an earlier approved stage, stop.
   Do not silently repair it. Propose a `REVISION` stage.
5. Prefer compilation errors, TODOs, and commented-out call sites over
   half-implemented logic.
6. Keep every stage small enough that the human can review it in under
   2–5 minutes.
7. Do not create tests automatically unless the human asks for them.
8. Do not refactor unrelated code.
9. Do not invent requirements, APIs, naming, or architecture.
10. If the task cannot be safely divided into these stages, say so and
    propose a custom stage plan first.

## Development comment format

Every new or changed declaration must carry a development comment:

```rust
/// DEV: [stage 2/state]
/// Purpose: caches the last validated user session.
/// Owner flow: refreshed by `SessionManager::refresh`, read by request guards.
/// Open question: should expired sessions be retained for audit?
/// TODO(stage 7): implement refresh and expiry logic.
pub last_valid_session: Option<Session>,
```

Required fields:

- `DEV`: stage and anatomical layer.
- `Purpose`: why this exists.
- `Owner flow`: who creates, mutates, reads, or removes it.
- `Open question`: uncertainty the human should resolve, if any.
- `TODO(stage N)`: the future implementation point, if any.

For non-doc-comment languages, use the project’s normal comment syntax,
but preserve the same fields.

## Stage plan

Before any code, produce a **Change Manifest**:

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

The human must approve the manifest before Stage 1.

### Stage 1 — Scaffold

Create only empty structures:

- new files,
- empty modules,
- empty structs/classes/enums/traits/interfaces,
- empty functions or methods,
- type placeholders where unavoidable.

No fields. No logic. No call sites.

Each item gets a development comment explaining its intended role.

End with:

```text
SCAFFOLD MAP
- <entity>: <purpose>
- <entity>: <purpose>

Awaiting approval for stage 1: scaffold.
```

### Stage 2 — State

Add fields, constants, configuration, internal state, and data structures.

Rules:

- No methods except trivial constructors if required to compile.
- No external behavior.
- Every field must have a `DEV` comment.
- Mark every field that will later require synchronization, validation,
  persistence, migration, or cleanup.

End with a **State Map**:

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

Await approval.

### Stage 3 — Integration points

Identify where the new state or entities will connect to existing code.

Allowed changes:

- imports,
- trait/interface implementations with empty bodies,
- registration points,
- dependency injection,
- configuration wiring,
- commented-out future call sites.

If a call site cannot compile or execute yet, leave it commented:

```rust
// DEV: [stage 3/call-site]
// TODO(stage 5): enable after SessionValidator::validate exists.
// session_validator.validate(&request)?;
```

End with an **Integration Map**:

```text
INTEGRATION MAP
- Existing component -> New component
  - direction of dependency
  - event / call / data flow
  - activation condition
  - current status: wired / commented / deferred
```

Await approval.

### Stage 4 — Signatures

Add function and method signatures.

Rules:

- Use precise parameter and return types.
- Add doc comments describing contract, preconditions, errors, and side effects.
- Bodies may only contain `unimplemented!()`, `todo!()`, `panic!("TODO")`,
  `throw new Error("TODO")`, or the project’s equivalent.
- Do not add temporary fake return values unless required to compile;
  clearly mark them as temporary.

End with a **Contract Map**:

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

Await approval.

### Stage 5 — Call sites

Add the places that invoke the new functions or use the new state.

Rules:

- Enable previously commented call sites only when their dependencies exist.
- Keep each call site minimal.
- Do not implement surrounding business logic.
- If error handling is needed, add explicit TODO handling rather than
  silently swallowing errors.

End with a **Call-site Map**:

```text
CALL-SITE MAP
- caller -> callee
  - trigger condition
  - arguments
  - expected result
  - failure behavior
  - current status
```

Await approval.

### Stage 6 — Existing-type changes

Modify existing types only now:

- new methods,
- new trait implementations,
- changed visibility,
- added variants,
- changed fields,
- interface adjustments.

Every modification needs:

```text
DEV: [stage 6/existing-change]
Before: <old behavior or absence>
After: <new behavior or responsibility>
Why: <reason>
Blast radius: <known affected callers>
```

End with a **Blast-radius Map**.

Await approval.

### Stage 7 — Bodies

Implement function bodies one function or small cohesive group at a time.

Rules:

- Start with the simplest dependency-free function.
- Do not implement multiple unrelated bodies in one stage.
- If a body requires a design decision, stop and ask.
- If a body reveals a bad earlier decision, propose `REVISION`, do not patch silently.
- Keep the diff focused.

For each implemented body, report:

```text
BODY REPORT
- function
- approach
- edge cases handled
- edge cases deferred
- assumptions
- follow-up TODOs
```

Await approval after each meaningful body or small group.

### Stage 8 — Verification

Only now propose verification:

- compile,
- lint,
- run relevant tests,
- manually exercise the flow,
- inspect logs or generated artifacts.

The agent must not claim success without evidence.

End with:

```text
VERIFICATION REPORT
- commands run
- expected result
- actual result
- failures and interpretation
- remaining risks
```

Await final approval.

## Review ritual

After every stage, the agent must output:

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

**Next stage**
- <name and one-sentence description>

Awaiting approval for stage N: <name>.
```

The human may reply with:

- `approve` — proceed to the next stage.
- `approve with notes: ...` — proceed, incorporating the notes.
- `revise: ...` — remain in the same stage and fix the issue.
- `replan: ...` — return to the Change Manifest.
- `stop` — halt and summarize current state.

## Commit ritual

Stages are committed incrementally so each gate leaves a restorable point,
but history stays squashable into one reviewable commit:

1. After the first approved stage (or first approved group of stages), create
   the **base commit** for the feature. Use the project's commit conventions.
2. After every subsequent stage approval, commit that stage's diff as a NEW
   fixup of the base commit — `git commit --fixup <base-sha>` — BEFORE starting
   the next stage. Never mix two stages in one fixup.
3. NEVER amend or rewrite already-created commits (neither the base nor any
   fixup). Post-approval revisions to an earlier stage are committed as their
   own additional fixup of the base commit. History only moves forward until
   the final squash.
4. Development comments (`DEV:` markers) may ride along in fixups; they are
   stripped in the verification stage's final fixup.
5. Before the PR, squash with `git rebase -i --autosquash` onto the base
   commit unless the human asks to preserve the stage history.
6. Never commit without the human's standing or explicit consent; the staged
   commit ritual itself must be approved once per feature.

## Entertainment mode

To keep the process engaging, end each stage with a short **Developer’s Log**:

```text
> Stage 3 complete.
> The skeleton now has nerves, but no muscles.
> Next: give the functions their signatures and promises.
```

Keep it to one or two sentences. Do not let flavor text replace technical review.

## Completion checklist

Before declaring done, confirm:

- [ ] Change Manifest matches the final implementation.
- [ ] Every new entity has a purpose comment.
- [ ] Every state field has an owner and lifecycle explanation.
- [ ] Every TODO is assigned to a stage or explicitly deferred.
- [ ] No commented-out code remains unless intentionally retained.
- [ ] All call sites are either active or explicitly disabled with a reason.
- [ ] Verification evidence exists.
- [ ] The human has approved the final state.
