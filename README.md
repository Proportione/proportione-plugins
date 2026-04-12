# Proportione Plugins — Claude Code Playbook

How a 1-3 person IT consultancy uses Claude Code to manage 13 WordPress sites, 4 GCP microservices, a scientific monorepo, and IaC infrastructure — with quality.

## What's here

This repo is both a **Claude Code plugin marketplace** (installable skills) and a **playbook** of everything we've learned using Claude Code in production since early 2026.

### Plugins (9 QA skills)

Install the marketplace, then pick what you need:

```bash
claude plugin marketplace add Proportione/proportione-plugins
claude plugin install deep-review@proportione-plugins
```

| Plugin | What it does |
|--------|-------------|
| `deep-review` | Deep code review for PRs, commits, or files — logic, naming, complexity, tests, security |
| `check-security` | OWASP-based security audit with Semgrep and manual analysis |
| `check-architecture` | Validate code against architectural conventions defined in CLAUDE.md |
| `check-performance` | Performance analysis: N+1 queries, bundle size, Web Vitals, Lighthouse, GCP sizing |
| `check-wordpress` | Full WordPress QA: accessibility, SEO, links, images, security, performance |
| `check-wp-visual` | Visual regression testing for WordPress sites using Playwright |
| `check-terraform` | IaC validation: terraform validate/plan + Checkov + Semgrep |
| `triage-bug` | Automated bug triage: error analysis, root cause, fix, and PR creation |
| `update-whitepaper` | Monthly QA whitepaper update with metrics collection and source research |

### Playbook (guides)

Hard-won lessons from running Claude Code in a real production environment:

| Guide | Topic |
|-------|-------|
| [CLAUDE.md as Contract](guides/claude-md-as-contract.md) | How we use CLAUDE.md as an architectural contract, not documentation |
| [MCP Servers](guides/mcp-servers.md) | Building MCP servers for ticketing, CRM, BI — and keeping auth working |
| [Playwright Tricks](guides/playwright-tricks.md) | Automating Google services, Synology DSM, Looker Studio, and more |
| [QA Framework](guides/qa-framework.md) | The five-layer QA model for AI-generated code |
| [Operational Rules](guides/operational-rules.md) | Governance rules for Claude Code in automation pipelines |
| [GCP Patterns](guides/gcp-patterns.md) | Cloud Run, Firestore, BigQuery patterns we use daily |
| [Sales & Marketing](guides/sales-marketing-with-claude.md) | CRM, prospecting, content, social media — all from the terminal |

## Our stack

- **WordPress** (13 sites): Elementor Pro + SiteGround
- **Backend**: Node.js 20 (Cloud Run), Python 3.14 (FastAPI), Terraform + Ansible
- **GCP**: Cloud Run, Firestore, BigQuery, Cloud Tasks, Vertex AI
- **QA tools**: Semgrep, Checkov, Lighthouse, Playwright, axe-core
- **IA stack**: Claude Code (architect) + Qwen 2.5 Coder 14B local (routine tasks)

## Why publish this

We wrote a [whitepaper](guides/qa-framework.md) arguing that AI-generated code needs AI-assisted QA. These are the tools we built to back that argument. If you run a small team writing code with Claude, these skills and patterns might save you time.

## License

MIT

---

Built by [Proportione](https://proportione.com) — strategy, technology, and people.
