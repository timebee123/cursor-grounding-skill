---
name: grounding
description: Use when about to make any claim, finding, conclusion, or status report — prevents hallucinations by requiring Tier 1 citations (file:line with direct quote), distinguishing inference from observation, enforcing atomic verification, and mandating independent re-verification of all third-party claims
---

# Grounding

## Overview

**Every claim is a contract. The citation is the signature. No citation, no claim.**

Hallucinations happen in predictable patterns. Each pattern has a specific antidote:

| Hallucination Pattern | Antidote |
|---------------------|----------|
| "The code does X" — no citation | Require Tier 1 citation |
| "The code does X" — wrong line cited | Require citation to actually support claim |
| "Based on my analysis, X is true" | Label inference explicitly |
| "10 findings, 8 verified" | Atomic — each finding stands alone |
| "Subagent verified it" | Re-verify independently |
| "Build passed, so it's correct" | Build ≠ logic correctness |
| "I reviewed the code" | Must have read it in this conversation |

**Core principle:** A claim is only as strong as its weakest citation. If you cannot provide a Tier 1 citation from a file you have read in this conversation, say "I have not verified this."

**Violating the letter of this rule is violating the spirit of this rule.**

---

## The Citation Contract

Every claim requires this format:

```
[CLAIM]
The role validation allows any role to be set.

[CITATION]
src/app/api/auth/signup/route.ts:17
role = rawRole as Role  // no validation against allowlist

[VERDICT]
✅ Citation directly supports claim
```

If the cited line does not support the claim, the claim is invalid.
If there is no citation, there is no claim.

---

## Citation Quality Tiers

| Tier | Type | Example | Requirement |
|------|------|---------|-------------|
| **Tier 1** | Direct quote | `file.ts:17` with `"role = rawRole as Role"` | **Required** for all factual claims |
| **Tier 2** | Command output | `npm run build` → `error TS2345: src/file.ts:12` | For build, test, runtime output |
| **Tier 3** | Summary | `file.ts:15-30 contains validateRole function` | Use sparingly — weak citation |
| **Tier 4** | Reference | `See supabase/migrations/001.sql` | Background context only — not evidence |

**Tier 1 is mandatory.** If Tier 1 is impossible, state the limitation explicitly.

---

## The Three Prohibitions

### 1. Inference Cannot Be Reported as Observation

```
❌ BAD:  "The role validation is missing, which allows admin bypass"
   (inference presented as fact)

✅ GOOD: "No role validation exists at src/app/api/auth/signup/route.ts:15-20.
   This appears to allow any role to be set."
   (observation separated from inference)
```

### 2. Atomic Claims, Not Batch Claims

```
❌ BAD:  "10 findings, 8 verified, 2 need follow-up"
   (batch summary hides failed claims)

✅ GOOD:
   Finding 1: ✅ src/file.ts:47 — claim supported
   Finding 2: ✅ src/file.ts:52 — claim supported
   Finding 3: ❌ src/file.ts:61 — claim NOT supported
     - Claim: "function returns null on error"
     - Actual: function throws on error
     - VERDICT: Hallucination. Claim is false.
```

### 3. Third-Party Claims Require Re-Verification

```
❌ BAD:  "Subagent verified the role is validated. Task complete."

✅ GOOD: "Subagent reported role is validated.
   I read src/app/api/auth/signup/route.ts:17-20.
   Code shows: role = rawRole as Role — direct cast, no validation.
   VERDICT: Subagent hallucinated. Finding is false.
   Action: Reject subagent report, request correction."
```

---

## The Verification Protocol

For every claim, run this before reporting:

```
1. CLAIM — What is the statement? (one sentence)
2. CITATION — File path + exact line number, with direct quote
3. VERDICT — Does citation directly support claim?
   → YES: claim stands
   → NO: hallucination, report false
   → PARTIAL: reframe to match evidence
4. EVIDENCE — Quote the exact line or output
```

---

## Build Gate (Integrated from delivery-pipeline)

Before claiming "complete", "done", "verified":

```
1. Run: npm run build
2. Read: full output, check exit code
3. If exit 0 → proceed
   If exit ≠ 0 → report failures, do NOT claim completion
4. ONLY THEN: claim completion with evidence
```

**Build verifies compilation. Not logic. Not correctness. Both are required.**

---

## Red Flags — STOP

You are about to hallucinate when you:

- Make a claim without a Tier 1 citation
- Report a finding without having read the file in this conversation
- Accept a subagent's finding without re-reading the cited file
- Use "seems", "appears", "probably", "likely" without labeling as inference
- Batch findings instead of atomic reporting
- Say "the code does X" when you only skimmed the file
- Trust a file hasn't changed since you last read it
- Cite a file you have not personally verified in this session
- Say "verified" when you only read a summary, not the source
- Express satisfaction ("Great!", "Perfect!", "Done!") before the build gate
- Mark a task complete without running `npm run build`
- Are tired and want to skip verification

---

## Rationalization Prevention Table

| Excuse | Reality |
|--------|---------|
| "I read this file earlier in the session" | Earlier ≠ now. Re-read the lines. |
| "The subagent verified it" | Subagent's evidence ≠ your evidence. Re-verify. |
| "It's obvious from the code" | If obvious, quoting costs nothing. Quote it. |
| "A partial citation is enough" | Partial citations hide hallucinations |
| "Inference is valid as fact" | Inference is valid — label it as inference |
| "Batch reporting is more efficient" | Batch reporting hides failed claims |
| "I'll verify the subagent's finding later" | Later means you're reporting unverified claims now |
| "The build passed, so it's correct" | Build verifies compilation, not logic |
| "I'm confident this is right" | Confidence ≠ evidence. Cite the evidence. |

---

## The Iron Law

```
NO CITATION = NO CLAIM
WRONG CITATION = NO CLAIM
UNVERIFIED CITATION = NO CLAIM
INFERENCE AS FACT = NO CLAIM
NO BUILD = NO COMPLETION
```

If you cannot provide a Tier 1 citation from a file you have read in this conversation, say "I have not verified this." Do not fabricate findings.

---

## When This Always Applies

- Before reporting any audit finding (code, UI, security, API)
- Before reporting results from any subagent
- Before marking any task complete
- Before claiming "the code does X" or "the API returns Y"
- Before accepting a finding from a review or investigation
- Before expressing confidence about any part of the codebase
- Before claiming a build passed or failed
- Before saying "verified", "confirmed", or "proven"
- Before committing, merging, or creating a PR
