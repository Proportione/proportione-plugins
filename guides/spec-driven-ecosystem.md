# Spec-Driven Development Ecosystem for Claude Code

A map of the tools and frameworks the community uses to validate specifications before writing code. Based on research and discussion with [@diegomarino](https://x.com/diegomarino), [@juanmacias](https://x.com/juanmacias), [@nacho540](https://x.com/nacho540), and others (April 2026).

## The problem

Claude Code is great at writing code, but it needs clear specs. Without them, it builds what it *thinks* you want — which is often close but not exactly right. The community has converged on "spec-first" as the standard approach, with different tools for different stages.

## Tool comparison

### GSD (Get Shit Done)
- **Repo**: [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done)
- **Install**: `npx get-shit-done-cc@latest`
- **What it does**: Full spec-driven development lifecycle — from project init to PR creation
- **Key commands**:
  - `/gsd-new-project` — Initialize with questions, research, requirements extraction, roadmap
  - `/gsd-discuss-phase` — Capture implementation preferences before building
  - `/gsd-plan-phase` — Create atomic task plans with verification steps (includes plan-checker)
  - `/gsd-execute-phase` — Run plans in parallel with fresh context per task
  - `/gsd-verify-work` — Test deliverables and diagnose failures
  - `/gsd-ship` — Create PRs from verified work
- **Strength**: Solves "context rot" — quality degradation as Claude fills its context window. Each task gets fresh context.
- **Plan-checker**: Integrated into `/gsd-plan-phase`. Checks plans against requirements, loops until they pass. Uses "Nyquist validation layer" to ensure test feedback loops exist before code is written.

### Superpowers
- **Repo**: [obra/superpowers](https://github.com/obra/superpowers)
- **Install**: `claude plugin marketplace add obra/superpowers-marketplace && claude plugin install superpowers`
- **What it does**: Composable skills framework with brainstorming, TDD, debugging, and collaboration
- **Key skills**:
  - `brainstorming` — Refines rough ideas through questions, explores alternatives, presents design in sections for validation
  - TDD skills — Ensure tests are written before code
  - Debugging skills — Methodical bug investigation
  - Personal skills — Create domain-specific skills that evolve with your workflow
- **Strength**: Skills are composable and Claude can create new skills itself. The system evolves.
- **Accepted into**: Official Anthropic plugin marketplace (Jan 2026)

### SuperClaude
- **Repo**: [SuperClaude-Org/SuperClaude_Plugin](https://github.com/SuperClaude-Org/SuperClaude_Plugin)
- **What it does**: Meta-programming framework with 29 `/sc:` commands, 23 specialized agents, 7 behavioral modes
- **Key commands**:
  - `/sc:spec-panel` — Spec validation panel (mentioned by [@nacho540](https://x.com/nacho540))
  - `/sc:brainstorm` — Ideation support
  - `/sc:analyze` — Codebase analysis
  - `/sc:research` — Deep research
- **Strength**: 23 specialized agents (security engineer, frontend architect, research agent) with automatic coordination based on context.

### Alfred Dev
- **Site**: [alfred-dev.com](https://alfred-dev.com/en/)
- **What it does**: 19 specialized agents, 60 skills, per-project persistent memory (SQLite)
- **Key commands**:
  - `/alfred-dev:feature` — Full development cycle up to 7 phases
  - `/alfred-dev:quick` — Small bounded changes with scoped QA + security validation
  - `/alfred-dev:fix` — Bug fixes in 3 phases (diagnosis, TDD correction, validation)
- **Agents**: data-engineer, ux-reviewer, performance-engineer, github-manager, seo-specialist, copywriter, The Librarian (persistent memory), The Interpreter (i18n)
- **Strength**: Persistent memory via SQLite — remembers decisions, commits, and iterations across sessions.
- **Status**: Alpha (mentioned by [@686f6c61](https://x.com/686f6c61))

### Proportione Plugins (this repo)
- **What it does**: 9 QA skills focused on post-implementation validation
- **Key difference**: We validate *after* code is written (code review, security, architecture, performance), not before. This complements the spec-first tools above.
- **Strength**: Multi-stack (Python, Node.js, PHP/WordPress, Terraform) and multi-domain (engineering, WordPress, IaC).

## How they fit together

```
Idea → Spec → Plan → Code → Review → Deploy
       ↑       ↑       ↑       ↑
    Superpowers GSD   Claude  Proportione
    SuperClaude Alfred  Code   Plugins
```

- **Superpowers / SuperClaude**: Brainstorming and spec refinement (pre-implementation)
- **GSD**: Planning with verification gates (pre-implementation)
- **Alfred Dev**: Full lifecycle with persistent memory (spec through deploy)
- **Proportione Plugins**: Post-implementation QA (code review, security, architecture, performance)

These aren't competitors — they're complementary. You can use Superpowers for brainstorming, GSD for planning, and Proportione plugins for post-merge QA.

## What we use at Proportione

We use a lighter approach:
1. **CLAUDE.md as contract** — defines what "correct" means for each project
2. **Spec in the ticket** — requirements live in the support ticket or issue
3. **`/deep-review`** — validates implementation against spec after coding
4. **`/check-architecture`** — ensures code follows project conventions
5. **`/check-security`** — catches security issues before deploy

We're evaluating GSD's plan-checker and Superpowers' brainstorming skill to add spec-validation *before* coding. The combination of pre-implementation spec validation + post-implementation QA skills would close the full loop.

## Installing the ecosystem

```bash
# Spec & planning (pick one or combine)
npx get-shit-done-cc@latest                                    # GSD
claude plugin marketplace add obra/superpowers-marketplace      # Superpowers
claude plugin install superpowers@superpowers-marketplace

# Post-implementation QA
claude plugin marketplace add Proportione/proportione-plugins
claude plugin install deep-review@proportione-plugins
claude plugin install check-security@proportione-plugins

# Diagrams & code quality
claude plugin marketplace add diegomarino/claude-toolshed
claude plugin install merge-checks@claude-toolshed
claude plugin install mermaid@claude-toolshed
```

---

*This guide was compiled from a [discussion thread](https://x.com/diegomarino/status/2043083991767474597) initiated by Diego Marino on April 11, 2026, with contributions from juanmacias, @686f6c61, @dobleio, @alex_cendal, and @nacho540.*
