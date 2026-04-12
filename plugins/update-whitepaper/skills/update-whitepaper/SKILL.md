# Skill: Update Whitepaper

Actualizacion mensual del whitepaper de QA y cumplimiento de requisitos de Proportione.

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)
- `$AUTOMATION_BRAIN_DIR` — Root of the automation-brain repo (e.g. `/path/to/automation-brain`)
- `$PORQUEVIVEN_DIR` — Root of the porqueviven-app repo (e.g. `/path/to/porqueviven-app`)

**Directorio:** `$QA_PROPORTIONE_DIR/whitepaper/`

## Flujo completo

### PASO 1 — Buscar nuevas publicaciones

Buscar publicaciones del ultimo mes en los blogs de referencia:

1. **Anthropic Blog** — buscar en https://claude.com/blog posts sobre Claude Code, code review, QA
2. **Claude Code Docs** — buscar cambios en https://code.claude.com/docs/en/best-practices y /code-review
3. **Anthropic News** — buscar en https://www.anthropic.com/news anuncios relevantes
4. **Addy Osmani** — buscar en https://addyosmani.com/blog/ posts sobre AI coding quality
5. **Hacker News** — buscar "claude code QA" o "AI code quality" en https://news.ycombinator.com/

Para cada publicacion relevante encontrada:

```
WebSearch: "[blog] claude code QA testing [mes actual] [ano]"
```

### PASO 2 — Crear extractos de nuevas fuentes

Para cada publicacion relevante:

1. Leer el contenido con WebFetch
2. Extraer los puntos clave relevantes para QA y cumplimiento de requisitos
3. Crear fichero en `whitepaper/fuentes/extractos/YYYY-MM-titulo.md` con formato:
   ```markdown
   # [Titulo]
   - **URL:** [url]
   - **Fecha publicacion:** [fecha]
   - **Acceso:** [fecha de hoy]
   ## Puntos clave
   [lista estructurada]
   ## Relevancia para Proportione
   [como aplicar a nuestro contexto]
   ```
4. Actualizar `whitepaper/fuentes/README.md` anadiendo la nueva fuente a la tabla

### PASO 3 — Ejecutar skills QA y recoger metricas

Ejecutar los skills disponibles contra los proyectos principales:

#### 3.1 Semgrep (si esta instalado)
```bash
which semgrep && semgrep --config $QA_PROPORTIONE_DIR/rulesets/semgrep/ \
  --json $AUTOMATION_BRAIN_DIR/ 2>/dev/null | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d.get('results',[])))"
```

Repetir para cada proyecto con codigo fuente.

#### 3.2 WordPress (Lighthouse + links)
```bash
# Si lighthouse esta instalado
which lighthouse && lighthouse https://proportione.com --output json \
  --chrome-flags="--headless --no-sandbox" --only-categories=performance,accessibility 2>/dev/null
```

#### 3.3 Test coverage
```bash
# automation-brain / IITD
cd $AUTOMATION_BRAIN_DIR/clientes/iitd/tools/integraciones/alumnos && \
  npm run test:coverage 2>/dev/null | tail -5

# porqueviven-app
cd $PORQUEVIVEN_DIR && npm run test:coverage 2>/dev/null | tail -5
```

#### 3.4 Dependencias vulnerables
```bash
# Python
pip audit 2>/dev/null | tail -5

# Node.js (por proyecto)
cd [proyecto] && npm audit --json 2>/dev/null | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Vulnerabilities: {d.get(\"metadata\",{}).get(\"vulnerabilities\",{})}')"
```

### PASO 4 — Generar informe mensual de metricas

Crear `whitepaper/metricas/YYYY-MM.md` con los resultados del PASO 3:

```markdown
# Metricas QA — [Mes] [Ano]

## Metricas por proyecto
| Proyecto | Semgrep | Cobertura | Lighthouse perf | Lighthouse a11y | Links rotos | Deps vulnerables |
...

## Comparacion con mes anterior
| Metrica | Mes anterior | Este mes | Delta |
...

## Destacados
- [Mejora mas significativa]
- [Problema mas urgente]
- [Nueva practica incorporada]
```

### PASO 5 — Actualizar secciones vivas del whitepaper

Editar `whitepaper/WHITEPAPER.md`:

1. **Seccion 6 (Aplicacion por proyecto):** Actualizar estado de cada proyecto con datos reales
2. **Seccion 7 (Metricas):** Actualizar tabla con valores del mes actual y enlace al informe
3. **Seccion 9 (Lecciones aprendidas):** Anadir 1-2 lecciones nuevas basadas en la experiencia del mes
4. **Version:** Incrementar version minor (1.0 -> 1.1 -> 1.2...)
5. **Fecha:** Actualizar fecha de ultima modificacion
6. **Proxima revision:** Actualizar al primer dia del mes siguiente

### PASO 6 — Actualizar CHANGELOG

Anadir entrada en `whitepaper/CHANGELOG.md`:

```markdown
## vX.Y — YYYY-MM-DD

### Fuentes nuevas
- [titulo y URL de cada fuente nueva]

### Metricas
- [resumen de metricas clave]

### Cambios en el framework
- [skills nuevos o modificados]
- [herramientas anadidas]
- [lecciones incorporadas]
```

### PASO 7 — Presentar resumen

Mostrar al usuario:

```markdown
## Actualizacion mensual QA — [Mes] [Ano]

### Nuevas fuentes incorporadas
- [lista]

### Metricas destacadas
- [tabla resumen]

### Cambios en el whitepaper
- [diff summary]

### Proximos pasos sugeridos
- [acciones basadas en metricas]
```

Preguntar: "Apruebas los cambios para actualizar el whitepaper?"

---

## Notas

- No actualizar el whitepaper sin revision humana
- Si no hay publicaciones relevantes en un mes, solo actualizar metricas
- Las metricas que requieren herramientas no instaladas se marcan como "Pendiente"
- Si un blog ha dejado de publicar contenido relevante durante 3 meses, considerar eliminarlo de la lista de monitorizacion
- Guardar siempre el informe mensual en metricas/ aunque los valores no cambien — el historico es valioso
