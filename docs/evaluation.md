# Evaluation

Date: 2026-11-25. Tool: NVIDIA SkillEvaluator 1.2.0, SkillSpector 2.13.6, Gitleaks (version reported as unknown). Command: `skillevaluator validate skills/blackbox-qa --tiers 1 --no-llm -c -r cli,json`.

**Evaluated artifact:** `skills/blackbox-qa/` working tree based on commit `883b095`; `SKILL.md` SHA-256 `f84500eea37571342025f6d7290bb93170efaf167fc0c6e41c8012bd6f4e7863`. The README and artwork were not scored.

| Check | Result |
| --- | --- |
| Tier 1 | 9 of 9 applicable checks passed; 3 not applicable |
| Static quality | A, 100/100: correctness 100, discoverability 100, reliability 100, efficiency 100 |
| Security | SkillSpector 2.13.6 and Gitleaks passed; Gitleaks version was unknown. These static checks are not a complete security assessment. |
| PII, license, Unicode, code integrity | Passed; no code tests were executed or coverage measured |
| Harness profiles | Six reported as loadable (advisory profile checks, not live agent runs) |

The evaluator flagged the `templates/` directory as nonstandard, although this repository deliberately uses it for skill resources. The previous skill revision at commit `443bea5` scored B, 84.8/100 (correctness 85, discoverability 75, reliability 85, efficiency 100); its security scan was incomplete because generated campaign filenames were treated as missing references. A subsequent 100/100 revision also had this caveat; the current revision clarifies those references and passes the scan.

Tier 2 semantic deduplication, Tier 3 live agent evaluation, and LLM-backed security analysis were not run. There are no executable scripts in the skill, so script linting and code risk analysis were not applicable. A static quality score does not measure agent performance. The raw JSON report was written outside the repository.
