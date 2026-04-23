# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/).
Versionado según [SemVer](https://semver.org/lang/es/).

## [Unreleased]

## [0.1.0] — 2026-04-23

Primera release etiquetada. Consolidación del marketplace tras tres meses de uso interno en Proportione.

### Added

- **9 plugins** (`plugins/`):
  - `deep-review` — revisión profunda de PRs, commits o ficheros (lógica, naming, complejidad, tests, seguridad).
  - `check-security` — auditoría OWASP con Semgrep + análisis manual.
  - `check-architecture` — validación contra convenciones en CLAUDE.md.
  - `check-performance` — análisis de rendimiento: N+1, bundle size, Web Vitals, Lighthouse, dimensionado GCP.
  - `check-wordpress` — QA completa WordPress: accesibilidad, SEO, enlaces, imágenes, seguridad, rendimiento.
  - `check-wp-visual` — regresión visual para sitios WordPress con Playwright.
  - `check-terraform` — validación IaC: terraform validate/plan + Checkov + Semgrep.
  - `triage-bug` — triaje automatizado de bugs: análisis, causa raíz, fix, apertura de PR.
  - `update-whitepaper` — actualización mensual del whitepaper QA con recolección de métricas.

- **8 guías** (`guides/`):
  - CLAUDE.md como contrato arquitectónico.
  - Construcción de MCP servers para ticketing, CRM, BI.
  - Playwright en Google, Synology DSM, Looker Studio.
  - Marco QA de cinco capas para código generado por IA.
  - Reglas operacionales para Claude Code en automatización.
  - Patrones GCP: Cloud Run, Firestore, BigQuery, Cloud Tasks, Vertex AI.
  - Ventas y marketing desde la terminal (CRM, prospección, contenido, redes).
  - Ecosistema *spec-driven*: GSD, Superpowers, SuperClaude, Alfred Dev.

- **Gobernanza**:
  - `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`.
  - Plantillas de Issue (bug, feature, question).
  - Plantilla de Pull Request.
  - Licencia MIT.

### Changed

- README con badges, one-liner de instalación destacado y sección de comunidad.

### Infrastructure

- Marketplace validado con `claude plugin marketplace add Proportione/proportione-plugins`.
- Topics GitHub: `claude-code`, `claude-code-plugin`, `claude-code-marketplace`, `mcp`, `ai-agents`, `qa`, `wordpress`, `gcp`, `playwright`, `semgrep`, `terraform`, `proportione`.
- Dominio de la organización verificado (`proportione.com`).

[Unreleased]: https://github.com/Proportione/proportione-plugins/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/Proportione/proportione-plugins/releases/tag/v0.1.0
