# blackbox-qa-skill

An [Agent Skill](https://agentskills.io) that runs a large black-box QA campaign against a built tool (MCP server, CLI,
library) on real example projects, using many agents with clear roles to keep cost and noise down. It encodes the
workflow used to QA `codenav-go-mcp`: ~20 areas, hundreds of cases, dozens of confirmed bugs.

It is a `SKILL.md` plus templates, written for Claude Code (subagent roles, worktrees) and adaptable to other agents that load skills.

## How it works

| Step | Who | Output |
|---|---|---|
| 1. Plan | one Opus agent | `qa/README.md`, `qa/<area>/checks.md` |
| 2. Workspace | orchestrator | `qa/<area>/bed/` (driver, `mkbed.py`, beds) |
| 3. Run | one Sonnet/Haiku tester per area; Opus advisor via the orchestrator, rarely | ticked checks, `bugs.md` |
| 4. Consolidate | orchestrator | `qa/BUGS.md` by severity |
| 5. Verify | narrow cheap verifiers, then one Sonnet reviewer | confirmed / not a bug / by design |
| 6. Fix (optional) | Sonnet splits bugs into parallel blocks; Haiku fixes each; Sonnet reviews each block | `qa/FIXPLAN.md`, patches |
| 7. Archive | orchestrator | keep text and scripts, trash beds and binaries |

## Install

### Any agent: the `skills` CLI

```bash
npx skills add illescasDaniel/blackbox-qa-skill
```

It asks which agents to install for and whether to install into the current project or globally. Non-interactive:
`npx skills add illescasDaniel/blackbox-qa-skill -a claude-code -y`, plus `-g` for global.

### By hand

Copy the `skills/blackbox-qa` folder into your agent's skills directory.

| | Project (shared with your team) | All your projects |
|---|---|---|
| Claude Code | `.claude/skills/blackbox-qa/` | `~/.claude/skills/blackbox-qa/` |
| Cursor | `.cursor/skills/blackbox-qa/` | `~/.cursor/skills/blackbox-qa/` |

## Contributing

Issues and pull requests are welcome. The skill is deliberately short: if a change makes `SKILL.md` longer, it should
earn it.

## License

MIT, see [LICENSE](LICENSE).
