# QA Framework for AI-Generated Code

> "When a human writes 50 lines a day, a human reviewer can keep up. When Claude Code generates 500 lines in 30 minutes, manual review becomes the bottleneck."

This is the framework Proportione uses to maintain quality in a codebase that's 100% AI-assisted.

## The problem

AI-generated code has three properties that break traditional QA:

1. **Volume**: 10x more code per session than manual development. Manual review can't keep up.
2. **Passes first read**: It looks correct, compiles, runs tests — but contains subtle logic/security/performance bugs that only emerge in production. Addy Osmani calls it the "overconfident junior developer" pattern.
3. **Inherits legacy lies**: When AI writes code on top of an existing codebase, it inherits and amplifies misleading variable names, misnamed methods, and leaky abstractions.

## The five-layer solution

### Layer 1: Architectural contract (CLAUDE.md)
Every project has a `CLAUDE.md` that defines conventions, folder structure, and rules. Claude reads it before every interaction. See [CLAUDE.md as Contract](claude-md-as-contract.md).

### Layer 2: Spec-first development
Write the specification before code. Claude validates implementation against spec, closing the requirements gap. Inspired by Addy Osmani's "waterfall in 15 minutes" approach.

### Layer 3: TDD loops
Claude writes code, runs tests, iterates until they pass. Tests are the agent's eyes — without them, it's blind. Anthropic's own teams use this pattern internally.

### Layer 4: Specialized QA skills
Nine skills (installable as plugins) that run targeted audits:

| When | Skill | Focus |
|------|-------|-------|
| Every PR | `/deep-review` | Logic, naming, complexity, edge cases, tests |
| Before production | `/check-security` | OWASP top 10 + Semgrep + dependency audit |
| Structural changes | `/check-architecture` | Drift from CLAUDE.md conventions |
| Performance PRs | `/check-performance` | N+1, bundle size, Web Vitals, GCP sizing |
| WordPress deploys | `/check-wordpress` | a11y, SEO, links, images, plugins, security |
| Visual changes | `/check-wp-visual` | Screenshot comparison against baselines |
| Infra changes | `/check-terraform` | terraform validate + Checkov + manual review |
| Bug reports | `/triage-bug` | Root cause → fix → PR, automated |

### Layer 5: Feedback loops
Skills improve over time. Each execution's lessons feed back into the skill's "notes" section. The `/update-whitepaper` skill runs monthly to collect metrics and track improvement.

## Theoretical foundations

This framework draws from five sources:

1. **Anthropic** — TDD loops and multi-agent review (5 parallel reviewers with different focuses)
2. **Qt Group** — AI augments QA roles, doesn't replace them. The human defines strategy; AI executes.
3. **Addy Osmani** — Spec-first development and testing as force multiplier
4. **juanmacias** — Iterative refinement achieving <1% error rate (Spanish developer, documented process)
5. **HAMY Labs** — Parallel subagent orchestration for complex reviews

## Key metrics we track

- Issues found by skill vs. found in production (should trend toward 100% pre-production)
- False positive rate per skill (should decrease as skills are refined)
- Time from bug report to fix (should decrease with `/triage-bug`)
- Architectural drift score (should stay low with `/check-architecture`)

## The "Capataz y Obrero" model

For routine tasks (style checks, boilerplate, test generation, documentation), we delegate to a local Qwen 2.5 Coder 14B model running on Ollama. Claude Code acts as the "capataz" (foreman) — planning, reviewing, making judgment calls. The local model acts as the "obrero" (worker) — executing routine tasks at ~25 tokens/second for ~2.16M tokens/day.

Estimated savings: ~130 EUR/month with 80% task delegation. GPU payback: 5-6 months.
