# Security Policy

## Supported versions

Este repositorio publica plugins y guías bajo licencia MIT. Cada release tagged recibe soporte hasta la siguiente release mayor.

| Versión | Estado |
|---------|--------|
| `main` (desarrollo) | ✅ soporte activo |
| `v0.x` últimas dos menores | ✅ soporte activo |
| Versiones anteriores | ❌ sin soporte |

## Cómo reportar una vulnerabilidad

No abras un Issue público para vulnerabilidades. Envíanos un correo a:

**[security@proportione.com](mailto:security@proportione.com)**

Incluye, en la medida posible:

- Plugin o guía afectada.
- Descripción del vector de ataque.
- Pasos de reproducción.
- Impacto potencial (lectura de datos, ejecución arbitraria, exfiltración).
- Versión(es) afectadas.

## Qué esperar

- Acuse de recibo en 48 horas laborables.
- Evaluación inicial y clasificación en 5 días laborables.
- Fix o plan de mitigación comunicado en 14 días para severidad alta/crítica.
- Divulgación coordinada tras el fix.

## Ámbito

Consideramos dentro del ámbito:

- Plugins y skills que se instalan con `claude plugin install`.
- Guías de `guides/` que contengan instrucciones ejecutables.
- MCP servers referenciados en el repositorio.

Queda fuera del ámbito:

- Vulnerabilidades en Claude Code en sí (reportar directamente a Anthropic).
- Vulnerabilidades en herramientas de terceros que usan las guías (Semgrep, Playwright, etc.).

## Reconocimiento

Publicaremos un agradecimiento en el changelog de la release correspondiente salvo que prefieras permanecer anónimo.
