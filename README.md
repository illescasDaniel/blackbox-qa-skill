<p align="center"><img src="assets/icon.png" alt="Black-box QA icon: a magnifying glass examining a cube" width="96"></p>

# Black-box QA

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

An [Agent Skill](https://agentskills.io) for planning and running a multi-agent black-box QA campaign against a built server, CLI, or library using real example projects. It organizes checks, bug reports, independent verification, and optional fixes.

## How it works

<img src="assets/workflow.png" alt="Campaign workflow: plan checks, build test beds, run testers, consolidate reports, verify bugs, optionally fix them, and archive the workspace" height="350">

A planning agent creates area checklists; testers run them against reproducible test beds. Reports are deduplicated and independently verified before any requested fixes. The workspace keeps the checks, reports, drivers, and scripts needed to reproduce the campaign.

## Quick start

```bash
npx skills add illescasDaniel/blackbox-qa-skill
```

Ask your agent to “QA this CLI on real projects” or “verify the reported bugs.” The skills CLI can install the skill for a supported agent and scope you select.

<details>
<summary>Manual installation</summary>

Copy `skills/blackbox-qa/` into the skills directory for your agent, for example `.claude/skills/blackbox-qa/` for a Claude Code project or `~/.agents/skills/blackbox-qa/` for Codex globally.

</details>

## Requirements and limits

The workflow assumes an agent environment able to delegate to subagents, with access to the tool and suitable example projects. Its model defaults name Claude model tiers; availability, cost, and delegation capabilities depend on the host. Parallel fixes also require git worktrees. Campaigns can be resource-intensive. Fixes are optional and require a user request; intended-behavior decisions go to the user.

## Evaluation

<img src="assets/evaluation.svg" alt="SkillEvaluator static quality A, 100 out of 100; security checks passed" width="920">

NVIDIA SkillEvaluator 1.2.0 scored the skill **A, 100/100** on its static quality checks on 2026-11-25. SkillSpector and Gitleaks checks passed; these static checks are not a complete security assessment. No live agent evaluation was run. See [evaluation details](docs/evaluation.md).

## Tests

The repository contains instructions and templates rather than executable scripts. There are no automated script tests; actual agent behavior and campaign outcomes remain untested for this revision.

## Contributing

Issues and pull requests are welcome. Keep the skill focused and its instructions concise.

## License

MIT, see [LICENSE](LICENSE).
