---
name: context-audit
description: Audit and restructure everything that loads into an agent's context automatically at the start of a session — CLAUDE.md and nested CLAUDE.md files, AGENTS.md, README sections written for agents, .cursorrules, .github/copilot-instructions.md, skill/rule/hook directories, and anything those files tell the reader to load up front. Measures session-start byte/token cost before and after, finds front-loaded reference material, duplicated or stale rules, and overconstrained instructions, then moves reference content into on-demand docs while keeping the main file lean. Use this whenever the user asks to "audit context," "clean up CLAUDE.md / AGENTS.md," "shrink the system prompt," "why is context so bloated," or similar — including via the explicit /context-audit invocation. Do not use it for ordinary feature work, even when that work happens to touch CLAUDE.md.
---

# Context Audit

Session-start context (CLAUDE.md, AGENTS.md, nested rule files, etc.) is loaded
into *every* conversation regardless of what the task actually is. Content that
is only relevant to a subset of tasks — schema tables, exhaustive file
inventories, past-bug lists, long runbooks — still gets paid for on every
single turn if it lives there. This skill finds that cost and moves it to
where it's loaded on demand instead, without losing any content and without
softening any rule that earns its keep.

Work through the steps below in order. Stop after Step 6 and let the user
review before committing anything — this skill produces a plan and a diff,
not an unsupervised commit.

## Scope

Find every file that loads into an agent's context automatically or by
convention at the start of a session, before doing anything else:

- `CLAUDE.md`, and any nested `CLAUDE.md` files in subdirectories
- `AGENTS.md`
- README sections written for agents rather than humans
- `.cursorrules`
- `.github/copilot-instructions.md`
- Skill, rule, or hook directories (e.g. `.claude/skills/`, `.claude/hooks/`)
- Anything one of the above files instructs the reader to load up front —
  that counts as session-start context too, even if it lives in a separate
  file.

## Step 1: measure before changing anything

Report the byte count of every file found above, and the total. Estimate what
that costs in tokens (roughly bytes ÷ 4 for English prose/code is a fine
estimate — say so if you're estimating). State the numbers plainly. This is
the number the rest of the audit exists to move, so get it on the record
before touching anything.

## Step 2: audit each file

For each file, look for these five things. They're not mutually exclusive —
one paragraph can be both front-loaded reference *and* stale.

1. **Front-loaded reference.** Schema tables, API rate limits, exhaustive file
   inventories, past-bug lists, runbook detail, long CI/CD catalogs — anything
   relevant to a subset of tasks but currently loaded for all of them.

2. **Duplication.** Rules restated across two or more files. Note which copy
   is more accurate — they usually drift apart over time, and the older or
   less-referenced one is the one that's typically stale.

3. **Overconstrained guidance.** Rigid procedural rules a current model
   handles better by judgment, and rules that now contradict the harness's
   own system prompt. Check specifically for "re-read the file after every
   edit to confirm it applied" — edit tools error on a failed write, and the
   Claude Code system prompt already tells the model not to re-read for that
   reason, so this instruction is now a net negative rather than neutral.
   Also flag instructions that forbid something the model no longer does by
   default — those just spend attention on a non-problem.

4. **Stale or contradictory claims.** Two files disagreeing on a fact, or a
   rule describing code that has since changed. Verify against the actual
   code before trusting either version — don't assume the more recently
   edited file is the correct one.

5. **Obvious-from-structure content.** Anything the model would learn faster
   by listing the directory or reading the type definition than by reading a
   prose description of it.

## Step 3: restructure

- The main instruction file (CLAUDE.md / AGENTS.md) becomes lightweight: what
  the repo is, the handful of genuine gotchas, the rules that must not be
  broken, and an index of what to load on demand. Aim for something a new
  human hire would actually read start to finish.

- Move reference material into a docs directory as topic files (e.g.
  `docs/context/*.md`). Index them in a table that says **when** to read each
  one, not just that it exists — "read this before touching the payment
  webhook," not "payment webhook details." Move the content; do not delete
  it.

- Give every rule exactly one canonical home. Where another file needs it,
  link to that home — don't restate it.

- Where two files disagreed in Step 2, verify against the code and fix
  whichever one was wrong.

- Favor pointing at real artifacts over describing them. A link to the actual
  config, test, or type definition beats a paragraph summarizing it, and a
  link can't drift out of sync with the code the way prose can.

## Step 4: what NOT to cut

Be conservative here — this step exists because Step 3's instinct is to trim,
and some content causes real damage if it's wrong or missing. Keep the
following inline, in full, no matter how long the file gets:

- Rules that already caused an incident, *and say so* — the incident is what
  makes the rule credible to a reader who wasn't there for it.
- Security, auth, permissions, and allowlist rules.
- Anything a hook, CI check, or lint rule mechanically enforces. If you
  reword it, confirm the enforcement still matches the new wording.
- Gates on destructive actions and on human approval.
- Explicit stated preferences about output format or workflow, even fussy
  ones — those are the repo owner's call, not a style question this skill
  gets to make.

If you think one of these should be cut or shortened anyway, raise it with
the user instead of removing it unilaterally.

## Step 5: verify

- Run whatever lint, format, link-check, or content-check tooling the repo
  has over the new and moved files. Report the actual output — don't claim
  it passed without having run it.
- Confirm nothing broke: section anchors other files link to, relative paths
  inside moved content, and any script or hook that greps these files by
  name or heading (e.g. a hook that looks for a specific `## Section` to
  enforce something).
- Read the seams where you split or spliced content — not just the diff
  stat — to catch a sentence that only made sense next to the paragraph it
  got separated from.
- Re-report the session-start byte/token total, before and after, using the
  same method as Step 1 so the numbers are comparable.

## Step 6: report

Present a summary with:
- Before and after size numbers (bytes and estimated tokens).
- A table of what moved where (old location → new location, one row per
  moved section).
- Every stale or contradictory thing found in Step 2, and how it was
  resolved.
- Anything deliberately left alone, and why (this should include everything
  from Step 4, plus anything you were unsure about and decided not to touch).

Then stop. Let the user review the plan/diff before anything is committed —
this skill's job ends at a reviewable result, not a pushed one.

## Ground rules

Do not water down a rule to make a file shorter. The goal is that the right
context arrives at the right time, not that the files are small. If a section
is long because the underlying thing is genuinely complicated, it stays long
— it just stops being loaded for every task regardless of relevance. When
genuinely unsure whether something is safe to move or trim, ask the user
rather than guessing.
