# CLAUDE.md as Architectural Contract

The single most impactful thing we did with Claude Code was treating `CLAUDE.md` not as documentation, but as a **binding contract** that Claude reads before every interaction.

## The pattern

Every project gets a root `CLAUDE.md` that defines:

1. **Folder structure** — where things go and why
2. **Naming conventions** — kebab-case, camelCase, PascalCase by stack
3. **Layer separation** — what can import what
4. **Mandatory files per service** — package.json, Dockerfile, deploy.sh
5. **Critical rules** — `.env` never committed, shared code in `comun/`, no cross-client imports

## Why it works

Claude Code reads `CLAUDE.md` at the start of every session. If the contract says "all filenames in kebab-case", Claude will name files in kebab-case. If it says "services live in `clientes/*/proyectos/servicios/`", Claude will create files there.

The precision of your CLAUDE.md directly determines the quality of Claude's output:
- **Vague CLAUDE.md** → vague, inconsistent code
- **Specific CLAUDE.md** → precise, convention-following code

## Real example: automation-brain

```markdown
# automation-brain

## Structure
- `clientes/{nombre}/` — one folder per client
- `comun/resources/` — shared knowledge base
- `comun/tooling/` — shared tools
- Names: `kebab-case`, lowercase only
- Each service: own `package.json`, `Dockerfile`, `deploy.sh`
- `.env` never committed — use `.env.example`

## Rules
- No imports between clients (clientes/A cannot import from clientes/B)
- Reusable tools go in `comun/tooling/`, not duplicated per client
- Client docs go in `clientes/*/docs/`
```

## Validation skill

We built `/check-architecture` to verify code against CLAUDE.md conventions automatically. It reads the contract, then checks every file against it — catching architectural drift before it compounds.

## Tips

- Be specific. "Use good naming" is useless. "All files in kebab-case, all React components in PascalCase" is actionable.
- Include the "why" for non-obvious rules. Claude can make better judgment calls with context.
- Update the contract when you intentionally diverge. A stale contract is worse than none.
- Keep it under 200 lines. Claude reads the whole thing every session — don't waste tokens on aspirational content.
- For multi-stack projects (we have Python, Node.js, PHP, Terraform), have one section per stack.
