# Cursor Grounding Skill

A Cursor AI agent skill that prevents hallucinations by requiring Tier 1 citations and enforcing evidence-based claims.

## What It Does

Every claim requires a citation. Every citation must match the claim. Inference must be labeled as inference.

```
[CLAIM]
The role validation allows any role to be set.

[CITATION]
src/app/api/auth/signup/route.ts:17
role = rawRole as Role  // no validation against allowlist

[VERDICT]
✅ Citation directly supports claim
```

## Citation Tiers

| Tier | Type | Example |
|------|------|---------|
| **Tier 1** | Direct quote | `file.ts:17` with `"role = rawRole as Role"` |
| **Tier 2** | Command output | `npm run build` → `error TS2345` |
| **Tier 3** | Summary | `file.ts:15-30 contains validateRole function` |
| **Tier 4** | Reference | `See supabase/migrations/001.sql` |

**Tier 1 is mandatory for all factual claims.**

## Installation

Copy `SKILL.md` into your Cursor agent skills directory:

```bash
mkdir -p .cursor/skills/grounding
cp SKILL.md .cursor/skills/grounding/
```

Or link from a central skills directory:

```bash
ln -s /path/to/grounding .cursor/skills/grounding
```

## The Iron Law

```
NO CITATION = NO CLAIM
WRONG CITATION = NO CLAIM
UNVERIFIED CITATION = NO CLAIM
INFERENCE AS FACT = NO CLAIM
NO BUILD = NO COMPLETION
```

## When to Use

- Before reporting any audit finding (code, UI, security, API)
- Before marking any task complete
- Before claiming "the code does X" or "the API returns Y"
- Before accepting a finding from a review or investigation
- Before claiming a build passed or failed
- Before committing, merging, or creating a PR

## Enforcement Layers

This skill works best with technical enforcement:

- **CI Gate**: GitHub Actions checks for uncommitted changes before merge
- **Pre-commit Hook**: Local hook validates skill exists before commit
- **Build Gate**: `npm run build` must pass before claiming completion

See [AGENTS.md](https://github.com/timebee123/cursor-grounding-skill/blob/main/SKILL.md) in your project for the full three-layer setup.

## Why This Exists

AI coding assistants hallucinate in predictable patterns:

| Hallucination | Antidote |
|--------------|----------|
| "The code does X" — no citation | Require Tier 1 citation |
| "The code does X" — wrong line cited | Require citation to support claim |
| "Based on my analysis, X is true" | Label inference explicitly |
| "10 findings, 8 verified" | Atomic — each finding stands alone |
| "Subagent verified it" | Re-verify independently |
| "Build passed, so it's correct" | Build ≠ logic correctness |

## License

MIT
