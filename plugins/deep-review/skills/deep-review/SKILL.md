# Skill: Deep Review

Revision profunda de codigo adaptada al contexto de Proportione.
Cada hallazgo se clasifica por severidad: CRITICAL, WARNING, SUGGESTION.

**Target:** $ARGUMENTS (PR number, commit range, file paths, o directorio)

## Configuration

This skill references external paths. Set these environment variables or replace inline:

- `$QA_PROPORTIONE_DIR` — Root of the QA_Proportione repo (e.g. `/path/to/QA_Proportione`)

## Flujo

### PASO 1 — Obtener el diff

Determina que revisar segun `$ARGUMENTS` o lo que el usuario proporcione:

```bash
# Si es un PR
gh pr diff [numero]

# Si es un rango de commits
git diff [commit1]..[commit2]

# Si es el working directory
git diff

# Si son ficheros especificos
# Leer los ficheros directamente
```

### PASO 2 — Leer contexto del proyecto

1. Lee el `CLAUDE.md` del proyecto para entender convenciones y arquitectura.
2. Identifica el stack (Python, PHP, JS/TS, Terraform, Ansible).
3. Lee `$QA_PROPORTIONE_DIR/CLAUDE.md` para contexto QA transversal.
4. Si existe documentacion de requisitos del cliente (specs, tickets, issues), leela para verificar cumplimiento.

### PASO 3 — Analisis profundo

Revisa cada fichero modificado evaluando:

#### Logica y correccion
- [ ] La logica es correcta para todos los inputs posibles?
- [ ] Se manejan los edge cases? (null, vacio, lista con 1 elemento, overflow)
- [ ] Las condiciones son completas? (falta algun else/case?)
- [ ] Los tipos/valores de retorno son consistentes?
- [ ] Hay race conditions o problemas de concurrencia?

#### Naming y claridad
- [ ] Las variables dicen lo que contienen?
- [ ] Los metodos hacen lo que dice su nombre?
- [ ] Hay variables con nombres enganosos? (el problema de codigo legacy que detecto juanmacias)
- [ ] Los nombres siguen las convenciones del proyecto?

#### Complejidad
- [ ] Se puede simplificar sin perder funcionalidad?
- [ ] Hay duplicacion que deberia extraerse?
- [ ] La funcion hace demasiadas cosas? (Single Responsibility)
- [ ] Los condicionales anidados se pueden aplanar?

#### Tests
- [ ] Existen tests para el codigo modificado?
- [ ] Los tests cubren los edge cases identificados?
- [ ] Si no hay tests, deberia haberlos?
- [ ] Los tests existentes siguen siendo validos tras el cambio?

#### Cumplimiento de requisitos
- [ ] El cambio implementa lo que se pidio? (ticket, issue, spec)
- [ ] Hay requisitos del cliente que el cambio no cubre?
- [ ] Se introduce comportamiento no solicitado?
- [ ] Los mensajes de error/UI cumplen con lo esperado por el usuario final?

#### Seguridad (check rapido)
- [ ] Se valida input del usuario antes de usarlo?
- [ ] Hay secrets hardcodeados?
- [ ] Se usa SQL parametrizado?
- [ ] Hay `eval()`, `exec()`, `dangerouslySetInnerHTML` o equivalentes?

#### Stack-especifico

**Python (Aviaria, automation-brain):**
- Se usan type hints en funciones publicas?
- Se usa `with` para file handles y conexiones?
- Hay `except Exception` genericos que enmascaran errores?

**PHP (WordPress):**
- Se usa `$wpdb->prepare()` para queries?
- Se escapan outputs con `esc_html()`, `esc_attr()`?
- Se verifican nonces en forms?
- Se usa `sanitize_*()` para inputs?

**JavaScript/TypeScript (porqueviven, OpenClaw):**
- Hay `any` types innecesarios?
- Se manejan promesas rechazadas?
- Se usa `===` en lugar de `==`?

**Terraform/Ansible:**
- Se parametrizan valores que pueden cambiar?
- Los recursos tienen tags/labels descriptivos?
- Se usa `depends_on` explicito cuando es necesario?

### PASO 4 — Generar informe

Presenta los hallazgos en este formato:

```markdown
## Deep Review — [nombre del PR/commit/ficheros]

### Resumen
[1-2 frases describiendo que hacen los cambios y la calidad general]

### Hallazgos

#### CRITICAL (bloquean merge)
1. **[fichero:linea]** — [descripcion del problema]
   - Impacto: [que puede pasar si no se corrige]
   - Fix sugerido: [codigo o descripcion del fix]

#### WARNING (deberian corregirse)
1. **[fichero:linea]** — [descripcion]
   - Fix sugerido: [...]

#### SUGGESTION (mejoras opcionales)
1. **[fichero:linea]** — [descripcion]

### Tests recomendados
- [ ] [test case 1]
- [ ] [test case 2]

### Veredicto
[APPROVE / REQUEST_CHANGES / NEEDS_DISCUSSION]
```

### PASO 5 — Ofrecer correcciones

Si hay hallazgos CRITICAL o WARNING, pregunta al usuario:

> Quieres que corrija los issues CRITICAL/WARNING directamente?

Si acepta, aplica los fixes y vuelve a ejecutar la revision para verificar.

---

## Notas

- Esta skill NO sustituye el juicio humano. Presenta hallazgos para que el humano decida.
- En codigo legacy, presta especial atencion a variables cuyo nombre no refleja su contenido real.
- Si el diff es muy grande (>500 lineas), divide la revision por fichero y presenta un resumen global al final.
- Cuando el proyecto tenga Semgrep configurado, ejecuta `semgrep --config $QA_PROPORTIONE_DIR/rulesets/semgrep/` como complemento.
