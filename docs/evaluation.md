# Evaluation

Date: 2026-11-25. Tool: NVIDIA SkillEvaluator 1.2.0, SkillSpector 2.13.6, Gitleaks (version reported as unknown). Command: `skillevaluator validate skills/blackbox-qa --tiers 1 --no-llm -c -r cli,json`.

**Evaluated artifact:** `skills/blackbox-qa/` at commit `443bea5`; `SKILL.md` SHA-256 `8c4e6fd5ec9ff699b05d2879efea7bc9099c4152a0e4fba2f41c0785aa052ed4`. The README and artwork were not scored.

| Check | Result |
| --- | --- |
| Tier 1 | 8 of 9 applicable checks passed; security incomplete; overall incomplete |
| Static quality | B, 84.8/100: correctness 85, discoverability 75, reliability 85, efficiency 100 |
| Security | SkillSpector scan incomplete: it reported unresolved file references, including paths for campaign-generated files. Gitleaks secrets detection passed; its version was unknown. This is not a complete security assessment. |
| PII, license, Unicode, code integrity | Passed; no code tests were executed or coverage measured |
| Harness profiles | Six reported as loadable (advisory profile checks, not live agent runs) |

The evaluator suggested shorter, earlier trigger wording, author and tags metadata, and explicit examples and requirements. It also flagged the `templates/` directory as nonstandard, although this repository deliberately uses it for skill resources. No previous-revision evaluation results were verified.

Tier 2 semantic deduplication, Tier 3 live agent evaluation, and LLM-backed security analysis were not run. There are no executable scripts in the skill, so script linting and code risk analysis were not applicable. A static quality score does not measure agent performance. The raw JSON report was written outside the repository.
