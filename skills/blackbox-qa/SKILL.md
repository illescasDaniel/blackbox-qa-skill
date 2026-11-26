---
name: blackbox-qa
description: Use when asked to QA a tool or verify reported bugs. Coordinates black-box testing on real projects and fixes confirmed bugs on request.
license: MIT
metadata:
  author: Daniel Illescas Romero
  tags: testing, qa, black-box, agents
---

# Black-box QA campaign

## Purpose

An expensive model writes the checklist, cheap models execute it on real projects and report bugs, and a cheap reviewer
confirms every bug before anything is fixed. Orchestrate; don't test yourself.

## Requirements

A built tool, its source and documentation, suitable example projects, git, and an agent host capable of delegation are required. Fixing in parallel also requires git worktrees. Record the tested revision, build command and tool version in the campaign README, named README.md in the generated QA directory.

## Instructions

### 1. Plan the checklist (Opus, once)

Spawn one Opus agent (planning only) to produce the campaign README and a checks.md file in each area directory (a tool or a
cross-cutting concern: protocol, concurrency, write safety, determinism, leaks, scale, environment). Give it the tool's
docs and source to read, and ask for suspected bugs found by reading the code ("run first" list).
Format: [templates/checks.md](templates/checks.md). Each case is `ID [P1|P2|P3] setup → call(s) → expected`, with P1 =
likely on real projects or loses data. Omit cases the unit tests already cover.

### 2. Build the workspace

- Add `qa/` to `.gitignore`. Under it, create one directory per area containing checks.md, bugs.md, and a bed directory. `bed/` holds the binary, driver, mkbed.py, logs and `beds/`.
- **Beds** = real projects to test on. Prefer offline sources (module/package cache, the language's stdlib), plus small
  synthetic beds with one directory per case. Each bed is a git repo with a clean base commit; preserve the original bed unchanged. Write cases always run on
  a **throwaway copy**; reset with `git checkout -- . && git clean -fdq`.
- A stdlib-only bed script (e.g. mkbed.py) must recreate every bed from scratch, so `beds/` and the binary can be deleted later.
- One small **driver** per area (e.g. raw stdio client that logs every call to log.jsonl) so cases are scripts, not
  hand-typed.
- The campaign README holds the ground rules, the bug format ([templates/bug.md](templates/bug.md)) and a per-area Status list. Ground rules apply to every case (e.g. "expected" always includes no absolute-path leaks, no raw stack
  traces, tool still responsive afterwards).

### 3. Run the testers (Haiku, one area each)

Launch one Haiku agent per area with [templates/tester-brief.md](templates/tester-brief.md). They tick each case
(`PASS` / `FAIL (BUG-<AREA>-nnn)` / `SKIP` with reason) and file bugs in bugs.md. They may message you with small questions (setup, how a tool works, is this a bug); answer
from the docs. If you can't, ask an Opus **advisor** a precise one-paragraph question and relay the answer. Do this rarely (a few times per campaign); testers must not call Opus themselves.
Run areas in parallel only if they don't share ports, files or a bed.

### 4. Consolidate

Merge all bugs into the campaign-wide BUGS.md in the QA directory: dedupe first (several areas often hit the same root cause; keep per-area ids and list duplicates), then sort by severity. Update
the Status list in the campaign README per area (cases, PASS/FAIL/SKIP counts, bug ids).

**Keep BUGS.md an index, not an archive** (see Context economy): full bug/root-cause entries go in one small file each
(e.g. `bugs/RC-001.md`), and BUGS.md keeps the summary, a one-line index table (id, severity, areas, status, title) and
pointers. Inline evidence in an entry only as the decisive lines; keep logs and transcripts on disk and reference paths.

### 5. Verify the bugs (second pass)

Verify in **batches of related bugs** (same area or same suspected root cause), never one agent per bug: spawn a Haiku
agent per batch with one narrow thing to confirm or refute
([templates/verifier-brief.md](templates/verifier-brief.md)): exact repro, expected vs actual, "is this by design?".
The verifier's prompt contains only its batch's bug entries and repro commands — it does not read the whole BUGS.md.
Then one **Sonnet reviewer** rules each bug CONFIRMED / NOT A BUG / BY DESIGN / DUPLICATE / INCONCLUSIVE, with severity:
it reads the verifier reports and the affected bug entries, not the full campaign record. Mark unreproducible or blocked
reports inconclusive rather than confirmed. Update the campaign-wide BUGS.md. Decisions about intended behaviour that
need the owner's judgment go to the user, not to an agent.

### 6. Fix (only if asked)

1. A **Sonnet planner** reads the campaign-wide BUGS.md and the per-area notes, groups the confirmed bugs into independent blocks
   (FIXPLAN.md in the QA directory), and writes a precise brief per block ([templates/fixer-brief.md](templates/fixer-brief.md)): bugs covered, files owned, decisions already made, ground
   rules (regression test per fix, build/lint/tests green, re-run the failing QA case). Blocks must not share files, so
   they can run in parallel. Open design questions go to the user or an Opus advisor, not to the fixers.
   **Make each brief self-contained**: the fixer reads only its brief plus the ground-rules block — never the whole
   BUGS.md or FIXPLAN.md. Keep FIXPLAN.md itself an index: one file per block brief, plus the map, waves and follow-ups.
2. One **Haiku fixer per block**, in parallel, each in its own git worktree. `qa/` is gitignored, so give fixers its absolute path (read-only except the patch). Each saves a patch to
   a blockNN.patch file in the QA fixes directory and reports what it changed.
3. After each block, a **Sonnet reviewer** checks the diff against the brief and the bugs: does it fix them, is the test
   real, any regression or scope creep. Small problems are fixed right then (send them back to the fixer or patch them);
   anything bigger or out of scope is recorded in FIXPLAN.md under that block as a follow-up.
4. Integrate the block patches one at a time, re-running build/lint/tests after each. Commit only when the user asks.

### 7. Archive

Keep checks, bugs, plans, patches, drivers and the bed script; delete `beds/`, copied binaries and big logs recoverably
(e.g. `trash`; `du -sh qa/*/bed/* | sort -h | tail` shows what weighs most). Note in the campaign README how to rebuild.

## Context economy

Lessons from a full campaign (~200 agent-runs): the biggest cost driver is not the tools or the code, it is agents
re-reading large campaign documents. Apply these throughout; they override nothing above, they just shape how
information is stored and handed out.

- **Indexes and per-id files.** Campaign-wide docs (BUGS.md, FIXPLAN.md, the README) are indexes: summary, one-line
  tables, pointers. Full entries live in one small file each (per root cause, per fix block). Agents open the files
  they were pointed at, never scan the indexes end-to-end.
- **Self-contained briefs.** Every agent prompt carries what that agent needs: its own entries, its owned files, the
  ground-rules block, the repro command. No agent's TODO starts with "read BUGS.md".
- **Batch verification.** One verifier per batch of related bugs (area or suspected root cause), not one per bug.
  ~150 single-bug verifier runs cost far more than ~30 batch runs and add little accuracy.
- **Reviewers read verdicts, not raw reports.** A reviewer gets the per-bug verdict lines plus the affected entries;
  it does not ingest every tester's and verifier's full transcript.
- **Evidence on disk, paths in context.** JSONL logs, diffs, bed copies and screenshots stay on disk (`qa/<area>/`,
  a logs dir); entries inline only the decisive lines and reference the paths. Cap inlined evidence to a few lines
  per bug.
- **Truncate at the source.** Tail/grep logs and test output before they reach any agent; a test suite's result is
  counts plus failing test names, not its raw output.
- **One short resume doc.** Mid-campaign pauses get a `LEFT.md` (open items, exact pointers, what not to redo). The
  next session reads that instead of reconstructing state from the whole record.
- **Pristine things stay pristine.** Throwaway copies for anything destructive; never rebuild a bed just to inspect
  it; delete scratch recoverably (`trash`), and say how to rebuild it in the README.

## Models

Defaults, chosen for cost (use available capability-equivalent agents if these models are unavailable): Opus plans the checklist and advises; Haiku tests, verifies and fixes; Sonnet orchestrates,
plans fix blocks and reviews. The model names used in the steps below are this shorthand, not requirements — pick the cheapest available model with the needed capability. The user may swap Haiku for Sonnet for testers or fixers (e.g. on subtle or large areas);
reviewers stay a tier above the agents they review.

## Examples

```text
QA this CLI on real projects; verify reported bugs and report the results.
```

## Limitations

This workflow requires independent agents and can consume substantial time and compute. Static checklists and verified reproductions do not establish complete coverage or prove the absence of bugs.

## Troubleshooting

If a bed, build, model, or environment is unavailable, record the blocked cases and reason as SKIP; do not claim a pass. If a verifier cannot reproduce a report, retain its evidence and mark it inconclusive pending a reproducible case.

## Rules

- Testers never edit the tool's source; fixers never edit `qa/` checklists.
- Never push, publish or hit the network unless the campaign needs it.
- A bug is not "confirmed" on one agent's word: step 5 comes before fixing.
