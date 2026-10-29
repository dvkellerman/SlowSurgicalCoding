# Slow Surgical Coding

An agent skill (for Cursor, Claude Code, and similar tools) that makes an AI coding agent build features in small stages, with your approval needed before each one, instead of writing everything in one big change.

You are the architect and reviewer. The agent works like a careful junior engineer: it proposes a change, explains it, and waits for your approval.

## How it works

1. **Change Manifest**: the agent first lists the goal, non-goals, new, changed, and removed entities, external effects, and risks.
2. **Eight stages**, each one reviewable in a few minutes:
   1. Scaffold
   2. State
   3. Integration points
   4. Signatures
   5. Call sites
   6. Existing-type changes
   7. Bodies
   8. Verification
3. **Review ritual**: every stage ends with a short diff summary, the reason for each change, any open questions, and a stop:
   `Awaiting approval for stage N: <name>.`

If a later stage reveals a flaw in a stage you already approved, the agent stops and proposes a `REVISION` stage instead of quietly fixing it.

## Installation

Copy `SKILL.md` into your skills directory, for example:

```bash
mkdir -p ~/.cursor/skills/slow-surgical-coding
cp SKILL.md ~/.cursor/skills/slow-surgical-coding/
```

## Usage

The skill activates on phrases like:

- "slow surgical coding"
- "one stage at a time"
- "go slow"
- "scaffold first"
- "staged implementation"

See [`SKILL.md`](SKILL.md) for the full rules.
