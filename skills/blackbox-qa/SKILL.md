---
name: blackbox-qa
description: Run a large, multi-agent black-box QA campaign against a built tool (server, CLI, library) on real example projects, then triage, verify and fix the bugs found. Use when asked to "QA this tool", "stress test it on real projects", "find bugs in my CLI/server/library", "dogfood this on real repos", "build a test checklist and have agents run it", or "verify the reported bugs".
---

# Black-box QA campaign

An expensive model writes the checklist, cheap models execute it on real projects and report bugs, and a cheap reviewer
confirms every bug before anything is fixed. Orchestrate; don't test yourself.

## 1. Plan the checklist (Opus, once)

Spawn one Opus agent (planning only) to produce `qa/README.md` and one `qa/<AREA>/checks.md` per area (a tool or a
cross-cutting concern: protocol, concurrency, write safety, determinism, leaks, scale, environment). Give it the tool's
docs and source to read, and ask for suspected bugs found by reading the code ("run first" list).
Format: [templates/checks.md](templates/checks.md). Each case is `ID [P1|P2|P3] setup → call(s) → expected`, with P1 =
likely on real projects or loses data. Omit cases the unit tests already cover.

## 2. Build the workspace

- Add `qa/` to `.gitignore`. Layout: `qa/<area>/{checks.md,bugs.md,bed/}`. `bed/` holds the binary, driver, `mkbed.py`, logs and `beds/`.
- **Beds** = real projects to test on. Prefer offline sources (module/package cache, the language's stdlib), plus small
  synthetic beds with one directory per case. Each bed is a git repo with a clean base commit. Write cases always run on
  a **throwaway copy**; reset with `git checkout -- . && git clean -fdq`.
- A stdlib-only bed script (e.g. `mkbed.py`) must recreate every bed from scratch, so `beds/` and the binary can be deleted later.
- One small **driver** per area (e.g. raw stdio client that logs every call to `log.jsonl`) so cases are scripts, not
  hand-typed.
- `qa/README.md` holds the ground rules, the bug format ([templates/bug.md](templates/bug.md)) and a per-area Status list. Ground rules apply to every case (e.g. "expected" always includes no absolute-path leaks, no raw stack
  traces, tool still responsive afterwards).

## 3. Run the testers (Sonnet or Haiku, one area each)

Launch one cheap agent per area with [templates/tester-brief.md](templates/tester-brief.md). They tick each case
(`PASS` / `FAIL (BUG-<AREA>-nnn)` / `SKIP` with reason) and file bugs in `bugs.md`. They may message you with a question (if the harness supports it); answer
from the docs. For a genuinely hard, narrow question, ask an Opus **advisor** a precise one-paragraph question and relay
the answer. Do this rarely (a few times per campaign); testers must not call Opus themselves.
Run areas in parallel only if they don't share ports, files or a bed.

## 4. Consolidate

Merge all bugs into `qa/BUGS.md`: dedupe first (several areas often hit the same root cause; keep per-area ids and list duplicates), then sort by severity. Update
the Status list in `qa/README.md` per area (cases, PASS/FAIL/SKIP counts, bug ids).

## 5. Verify the bugs (second pass)

For every bug (batch small ones per area), spawn a cheap agent with one narrow thing to confirm or refute
([templates/verifier-brief.md](templates/verifier-brief.md)): exact repro, expected vs actual, "is this by design?".
Then one **Sonnet reviewer** reads all verifier reports plus the original entries and rules each bug CONFIRMED /
NOT A BUG / BY DESIGN / DUPLICATE, with severity. Update `BUGS.md`. Decisions about intended behaviour that need the
owner's judgment go to the user, not to an agent.

## 6. Fix (only if asked)

1. A **Sonnet planner** reads `BUGS.md` and the per-area notes, groups the confirmed bugs into independent blocks
   (`qa/FIXPLAN.md`), and writes a precise brief per block ([templates/fixer-brief.md](templates/fixer-brief.md)): bugs covered, files owned, decisions already made, ground
   rules (regression test per fix, build/lint/tests green, re-run the failing QA case). Blocks must not share files, so
   they can run in parallel. Open design questions go to the user or an Opus advisor, not to the fixers.
2. One **Haiku fixer per block**, in parallel, each in its own git worktree. `qa/` is gitignored, so give fixers its absolute path (read-only except the patch). Each saves a patch to
   `qa/fixes/blockNN.patch` and reports what it changed.
3. After each block, a **Sonnet reviewer** checks the diff against the brief and the bugs: does it fix them, is the test
   real, any regression or scope creep. Small problems are fixed right then (send them back to the fixer or patch them);
   anything bigger or out of scope is recorded in `FIXPLAN.md` under that block as a follow-up.
4. Integrate the block patches one at a time, re-running build/lint/tests after each. Commit only when the user asks.

## 7. Archive

Keep checks, bugs, plans, patches, drivers and the bed script; delete `beds/`, copied binaries and big logs recoverably
(e.g. `trash`; `du -sh qa/*/bed/* | sort -h | tail` shows what weighs most). Note in `qa/README.md` how to rebuild.

## Rules

- Testers never edit the tool's source; fixers never edit `qa/` checklists.
- Never push, publish or hit the network unless the campaign needs it.
- A bug is not "confirmed" on one agent's word: step 5 comes before fixing.
- Opus only plans the checklist and advises; Sonnet/Haiku do everything else.
